RSpec built-in matchers

object.should be(expected) identity
object.should eq(expected) equivalence

all the same as
object.should ==
object.should eql
object.should equal

actual.should be > expected

actual.should be_within()

actual.should be_instance_of(expected)

actual.should be_true #(truthy - not nil or false)
actual.should be_false #(falsy - nil or false)
actual.should be_nil
actual.should be # same as be_true

expect {}.to raise_error(params)
expect {}.to throw_symbol

actual.should be_xxx # xxx is a method name, executes with truthy_result
actual.should have_xxx(:arg) #e.g. user.has(todo)

actual.should include(vals) #array values, hash key/val pair, string/partial

(1..10).should cover(3)

it { should have(3).items }
it { should_not have(2).items }
it { should have_exactly(4).items}
it { should have_at_least(5).items }

it { should respond_to(:length) } #can call method, doesn't have to be on class, can be on superclass

10.should satisfy { |v| v % 5 == 0 }

it { should be_kind_of(Fixnum) }

it { should end_with(string) }
it { should start_with(string) }


Writing controller tests with Rspec

Less of testing a unit, more of testing components in collaboration
Todo app - index, new, create

mkdir spec/controllers
touch todos_controller_spec.rb

require 'spec_helper'
describe TodosController do
  describe "GET index" do
    # get data ready - set instance variables
    # render template
    it "sets the @todos variable" do
      cook = Todo.create(name: "cook")
      sleep = Todo.create(name: "sleep")

      get :index
      assigns(:todos).should == [cook, sleep]
    end

    it "renders the index template" do
      get :index # need to trigger actual HTTP call for each test!
      response.should render_template :index
    end
  end

  describe "GET new" do
    it "sets the @todo variable" do
      get :new
      assigns(:todo).should be_new_record
      assigns(:todo).should be_instance_of(Todo)
    end

    it "renders the new template" do
      get :new
      response.should render_template :new
    end
  end

  describe "POST create" do
    it "creates the todo record when the input is valid" do
      post :create, todo: {name: "cook", description: "Yay cooking!"}
      Todo.first.name.should == "cook"
      Todo.first.description.should == "description"
    end

    it "redirects to the root path then the input is valid" do
      post :create, todo: {name: "cook", description: "Yay cooking!"}
      response.should redirect_to root_path
    end
    
    it "does not create a todo when the input is invalid" do
      post :create, todo: {description: "Yay cooking!"}
      Todo.count.should == 0
    end

    it "renders the new templatewhen the input is invalid" do
      post :create, todo: {description: "Yay cooking!"}
      response.should render_template :new
    end
  end
end

Cardinality and boundary conditions

When testing, follow 0, 1, many, boundary condition

Todos now has an input form, and tags. M to M association between todos and tags. 

class Todo
  has_many: taggings
  has_many: tags, through :taggings
end

class Taggings
  belongs_to :tag
  belongs_to :todo
end

Rule: if no tags, display todo name.
If tags, display inline (limit 4)

todo_spec.rb (model)

describe Todo do

  describe  "#display_text" do
    it "displays the name when there's no tags" do
      todo = Todo.create(name: "cook dinner")
      todo.display_text.should == "cook dinner"
    end

    it "displays the only tag with word "tag" when there's one tag" do
      todo = Todo.create(name: "cook dinner")
      todo.tags.create(name: "home")

      todo.display_text.should == "cook dinner (tag: home)"
    end
    
    it "displays the name with multiple tags" do
      todo = Todo.create(name: "cook dinner")
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")

      todo.display_text.should == "cook dinner (tags: home, urgent)"
    end

    it "displays no more than four tags" do
      todo = Todo.create(name: "cook dinner")
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")
      todo.tags.create(name: "help")
      todo.tags.create(name: "book")
      todo.tags.create(name: "patience")

      todo.display_text.should == "cook dinner (tags: home, urgent, help, book, more...)"
    end
  end
end

def display_text
  if tags.any?
    name + " (#{tags.one? ? 'tag' : 'tags' )}: #{tags.map(&:name).first(4).join(", ")} #{', more...' if tags.count > 4})"
  else
    name
  end
end

tags.map(&:name) #same as
tags.map { |tag| tag.name }

Resulting function is fairly complex, but result of building up through tests!

Refactoring in TDD

Official TDD cycle: red, green, refactor
Add features in the red, refactor in the green

our display_text method embeds a lot of logic
it's a public method, can be called outside of the class
but it's confusing if a programmer looks at it.

def display_text
  name + tag_text
end

private # only used by display_text

# could clean it up, but it's ok because not public interface now
def tag_text
  if tags.any?
    " (#{tags.one? ? 'tag' : 'tags' )}: #{tags.map(&:name).first(4).join(", ")} #{', more...' if tags.count > 4})"
  else
    ""
end

* Alternative style of Rspec

describe "#display_text" do
  let(:todo) { Todo.create(name: "cook dinner") }
  let(:subject) { Todo.display_text }

  context "no tags" do
    it "displays the name when there's no tags" do
      subject.should == "cook dinner"
    end
  end

  context "one tag" do
    before { todo.tags.create(name: "home") }
    it "displays the only tag with word "tag" when there's one tag" do
      subject.should == "cook dinner (tag: home)"
    end
  end

  context "multiple tags" do
    before do
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")
    end
    it "displays the name with multiple tags" do
      subject.should == "cook dinner (tags: home, urgent)"
    end
  end

  context "more than four tags" do
    before do
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")
      todo.tags.create(name: "help")
      todo.tags.create(name: "book")
      todo.tags.create(name: "patience")
    end
    it "displays up to four tags" do
      subject.should == "cook dinner (tags: home, urgent, help, book, more...)"
    end
  end
end

  context "no tags" do
    it { should == "cook dinner" }
  end

  context "one tag" do
    before { todo.tags.create(name: "home") }
    it { should == "cook dinner (tag: home)" }
    end
  end

  context "multiple tags" do
    before do
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")
    end
    it { should == "cook dinner (tags: home, urgent)" }
  end

  context "more than four tags" do
    before do
      todo.tags.create(name: "home")
      todo.tags.create(name: "urgent")
      todo.tags.create(name: "help")
      todo.tags.create(name: "book")
      todo.tags.create(name: "patience")
    end
    it { should == "cook dinner (tags: home, urgent, help, book, more...)" }
  end

# a lot of people prefer this style.
# pro: takes common setups from each test case, subject to assert against
# con: if you have a lot of test cases, the setup will be far from local code
# Kevin likes to start with the simple style, refactor if necessary

#P.S. don't need a let() for :subject line

#Lazy evaluation
Doesn't execute when it sees let, same for subject.
Created when before block runs

Don't want lazy evaluation? let!(:todo)

# The single assertion principle

For every test case, you should only have one assertion. 
Look out for the word "and" in your description! You're probably trying to do too much.

There are exceptions - in our controller spec, for example, should be_new_record, and should be_instance_of are asserting about the state of the same object. So treat this as a guideline, not an absolute law.

# Generating test objects via fabrication
We generate a lot of objects for tests, especially in setup.
What if, say, todo had 10 attributes, and 5 were required? That would be a real pain!

Recommended object generator framework: Fabrication
look at documentation

$mkdir spec/fabricators
todo_fabricator.rb

Fabricator(:todo) do
  name { "cooking" }
  user { Fabricate(:user) }
end

todo_spec.rb

describe "#display_text" do
  let(:todo) { Fabricate(:todo) }

  (Fabricate.build to make without saving to database)

# Generate fake data using Faker
Fabricators generate objects with same name.
What if there's, say, a unique user email validation in place?
Can use a fabrication sequence (see docs)
Alternately, install Faker (see docs)
faker::Name.name
faker::Lorem.words(5)

Fabricator(:todo) do
  name { Faker::Lorem.words(5).join(" ") }
end

Fabricator(:email) do
  email { Faker::Internet.email }
end

HW1
write two controller tests
video show
video search

HW2 
authentication tests
users controller
session controller
happy and bad paths! :D 

HW3 
enable reviews
sort in reverse chronological order
enable average rating
make sure to fabricate some reviews in your test

# match against array regardless of order
=~ 
or
match_array

rake db:reset 
Ask Kevin: talk a little bit about the switch to scope blocks for order

HW4
let the user submit reviews
test first
happy path, invalid path, user not logged in path


, url: {action "reviews#create"}, html: {class: "form-control"}

{"utf8"=>"✓",
 "authenticity_token"=>"sx309jLVLDdIrmeU8HcDSW+JQR4gNfeD4kR399ZviWY=",
 "review"=>{"rating"=>"5", "review_text"=>"pooooop"},
 "commit"=>"Submit",
 "action"=>"create",
 "controller"=>"reviews",
 "video_id"=>"1"}

 params.merge!
 map pluralize

 bad migration? if it was the last one, fix it, do 
 rake db:rollback db:migrate db:test:prepare

Now:
Implementing a queue
My Queue
Add a video to queue
Rating carries over to queue
Remove video with X
Re order videos by changing numbers around
Can also rate videos right in the queue
LATER: Implement video playing (in order)

Part 1
Build My Queue page
Show list of entries
Each item tracks video, current video, position number.
Focus on title, rating, genre columns first.
Remove, change rating, reorder implement later. (Phew... how to do rating, with review restrictions? New DB approach?)
Add video later

Fabricate queue items.
TDD process

Part 2 
Enable the add video ability for a user.
Make sure that the video is added to the bottom of the queue.

note
unless current_user.queue_items.map(&:video).include?(video)

I preferred to do it at the model level....

Part 3 
Remove video from queue
Redirect user back to my_queue