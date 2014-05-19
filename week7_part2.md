Beyond MVC - decorators

How to manage complexity as your application grows
Introduce techniques and patterns
When to use or not, tradeoffs involved

Decorator pattern

def display_text
  name + tag_text
end

This method is just to display todo with tags on webpage, purely presentation logic
Right now in model. Name of todo, plus tag_text (private method).

mkdir app/decorators
# everying in app directory is autoloaded by rails 

todo_decorator.rb:
class TodoDecorator
  attr_reader :todo
  def initialize(todo)
    @todo = todo
  end

  def display_text
    todo.name + tag_text
  end

  def decorator
    TodoDecorator.new(self)
  end

  private 

  def tag_text
    if todo.tags.any?
    etc
    else
    end
  end
end

Now in the view
= link_to TodoDecorator.new(todo).display_text, #todo
= link_to todo.decorator.display_text, #todo (with decorator method)

Don't want the view to have to remember whether to hit decorator or model

def index
  @todos = current_user.todos.map(&:decorator)
end

class TodoDecorator
  def_delegators :todo, :name_only?
end

Decorators are valuable when persist models one way in database, different than presented on webpage
Can test presentation logic separately
gem draper does decorators really easily
convenient methods like delegate all
convention over configuration - knows model article if decorator ArticleDecorator

HW
Create a decorator for video model using the draper gem
Video show template - showing a video's rating. Move this logic to the decorator.

Policy Objects
complicated logic in our create controller now - ways to clean up
different types of users when it comes to charging credit

model user_level_policy

class UserLevelPolicy
  attr_reader :user
  def initialize(user)
    @user = user
  end

  def premium? 
    user.created_at < Date.new(2010, 1, 1) || user.plan.premium?
  end
end

controller:
if UserLevelPolicy.new(current_user).premium?

Best - if you have bronze, silver, gold, etc. users - can define them all in one place - easy to find and change in one place.

Domain objects
We already have Domain objects - inherit from ActiveRecord::Base
If look at above code - user credit very important
In the code - always have to reach for user and get credit balance, manipulate, resave
Extract credit into its own model

model credit
class Credit
  attr_accessor :credit_balance
  def initialize(user)
    @credit_balance = current_user.credit_balance
    @user = user
  end

  def -(number)
    credit_balance = credit_balance - number
  end

  def save
    user.current_credit_balance = credit_balance
    user.save
  end

  def depleted?
    credit_balance < 0
  end

  def low_balance?
    credit_balance < 10
  end
end

def create
  @todo = Todo.new(params[:todo])
  credit = Credit.new(current_user)

  if @todo.save_with_tags
    if UserLevelPolicy.new(current_user).premium?
      credit = credit - 1
    else
      credit = credit - 2
    end

    credit.save

    if credit.depleted?
      AppMailer.notify_insufficient_credit(current_user).deliver
    elsif credit.low_balance?
      AppMailer.notify_low_balance(current_user).deliver
    end

    redirect_to root_path
  end
end

All business logic is in one model, for credit
And you're ready to save as a separate table if you want someday, just extend from ActiveRecord::Base
But the point is that you don't need to couple the objects directly with the way they're persisted.

Service Objects
All code inside create conditional is to determine how to handle credit in user's account
Use service objects to make business process more explicit

app/services/credit_deduction.rb

# some like verb, DeductCredit
# ok, but start to look odd if instantiated (DeductCredit.new)
# could do CreditDeductor or CreditManager
class CreditDeduction
  def initialize(user)
    @credit = Credit.new(user)
    @user = user
  end

  def deduct_credit
    if UserLevelPolicy.new(user).premium?
      credit = credit - 1
    else
      credit = credit - 2
    end

    credit.save

    if credit.depleted?
      AppMailer.notify_insufficient_credit(user).deliver
    elsif credit.low_balance?
      AppMailer.notify_low_balance(user).deliver
    end
  end
end

def create
  @todo = Todo.new(params[:todo])
  if @todo.save_with_tags
    CreditDeduction.new(current_user).deduct_credit
    redirect_to root_path
  else
    render :new
  end
end

Object composition, object-oriented design, YAGNI
Now, all the complex logic gets delegated to credit deduction service object.
That object is collaborating with a Credit object and UserLevelPolicy object, as well as AppMailer.

This is called object composition - use small components with simple responsibilities to compose complex workflow
Rails provides MVC, but need to do more as it grows. Initially just push down to model, but can eventually grow too bloated. Particularly models at center of domain, e.g. user model
Those are also called "God objects."
People are afraid to touch because easy to screw everything up.
This design is modular, reusable.
Cost: indirection - now need to chase down several objects to see what happens. 
May not be a problem, use good names.
But definitely possible to abuse:
Two parts to deduct credit, deduct credit and send email
Could extract each into service object, but that would be too much for now
May not be reused in the future.
YAGNI - You aren't going to need it
Don't prematurely extract for maintainability
You'll always know more in the future than now
Ruby is a flexible and dynamic language
In fact, looking at UserLevelPolicy, if that one-line premium method is all you have... might be better just putting it on the user model.
If it grows in the future, have multiple levels, complex logic for membership, that's when you want to extract it.
This is a really big topic, takes lots of learning and practice to get it right.

Message Expectations
Rspec message expectations
Expect that an object should receive a specific message
Have a logger that is a test double. 

it "logs an 'account closed' message" do
  logger = double()
  account = Account.new
  account.logger = logger
  
  logger.should_recieve(:account_closed).with(account)

  account.close
end

put the assertion in front of the trigger, a bit different from Rspec style so far
also should_receive and_return

Object.any_instance.should_receive(:foo).and_return(:return_value)

allow_message_expectations_on_nil
nil.should_receive(:foo)

can also assert how many times it receives a message

Mocking
How to write a test for the controller, with message expectations

Here the todos controller is collaborating with CreditDeduction - delegating task to instance of service object

Only need to test that the controller is telling the service object to deduct credit
The actual tests go in the CreditDeduction class

This is called "mocking"
So far have been asserting on values
Now, asserting on communication

Make object a test double, assert receives message

it "delegates to credit deduction to deduct credit" do
  credit_deduction = double("credit deduction")
  CreditDeduction.stub(:new).with(alice).and_return(credit_deduction)
  credit_deduction.should_receive(:deduct_credit)
  post :create, todo: {name: "cook", description: "I like cooking!"}
end

So here we first create a test double
Then we stub the new action, so returns a test double
The test double should receive a message, which is :deduct_credit
Can change logic inside credit deduction, but doesn't impact test in todos controller

Another example
Look at domain model Credit

for depleted? and low_balance? methods
query methods
set credit balance to specific value - return true or false depending on level

- method mutates credit object
pass in a number to method, assert that after invocation, credit object in a certain state (equals value)

lastly, save method
goes beyond boundary of current object, invoking methods on collaborator, user object
true unit test - mock collaborator user object with a test double
past test double into initialize
assert double received a message, current_credit_balance = 
user.should_recieve(:current_credit_balance).with(credit_balance)
user.should_receive(:save)

Mocks tricky when you get started, but very powerful when well used. Scope tests to object
Good for OO design skills - if have to assert several - too many collaborations - change your design.

Stubs and mocks
Mocks aren't stubs.
read article by Martin Fowler - read blog
watch "Why you don't get Mock objects" from rubyconf 2011

- Dummy objects are passed around but never actually used. Usually they are just used to fill parameter lists.
- Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example).
- Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test. Stubs may also record information about calls, such as an email gateway stub that remembers the messages it 'sent', or maybe only how many messages it 'sent'.
- Mocks are what we are talking about here: objects pre-programmed with expectations which form a specification of the calls they are expected to receive.

Talk
When your test is asserting on state, you're stubbing
When your test is asserting on messages being passed between objects, you're mocking
Alan Kay - Smalltalk founder - worry more about messages between objects than exact content. It's here that the meaning of the system is truly found.
Good OO design - tell the object to do certain things, rather than asking for things from it.
Hiding internal state. No getters. Good - this is how libraries work - don't need to know all the details behind every function. Moving this way with personal code is a good idea.
How to unit test? Adding getters just for that is bad.
Assert on the message that one system is telling another system, putting in a mock object

describe TicketMachineInterface do
  it "reserves the number of tickets input when the user submits a request" do
    request_handler = double('request_handler')
    request_handler.should_receive(:reserve).with('55')
    machine = TicketMachineInterface.new(request_handler)
    machine.number_pressed(5)
    machine.number_pressed(5)
    machine.submit_request
  end
end

In OOP, behavior found in messages between objects. Put in a mock to assert on messages
Takes a change in mindset. If procedural Rails... maybe not for you.
Key mocking rules - from Freeman and Pryce Java book
Mock roles, not objects
Wanting to mock concrete objects is a design smell
Well designed objects should only know the role they are playing
TDD becomes a design process - what doesn't belong in the object
Only mock types you own - if you don't own API, no feedback for test, you are testing an implementation
Duplicating production code in test is a smell
Refactor code example to display object - happens to use puts right now, but if switch to a GUI at some point, rest of game doesn't know difference 
How to test that? integration/acceptance


HW
implement a service object that handles the registration process
payment, invitation, email, notification
move a lot of controller tests to service object
controller test - assert delegating user signup to service object
