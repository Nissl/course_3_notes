Adding complexity guided by tests

Back to the todo app.
Add some complexity to application.
Smart enough to parse "study at home", understand location is home
Due to following "at"
tags(location: home, location: work)

So... adding complexity. What happens if mistake? How to recover?

Still testing "POST create"

context "with inline locations" do
	it "creates a tag with one location" do
		post :create, todo: {name: "cook at home"}
		expect(Tag.all.map{&:name).to eq('location:home')
		# equivalent of Tag.all.map(&:name) Tag.all.map { |tag| tag.name }
	end
end

First try, got nothing back, because nothing implemented yet
To satisfy

def create
	@todo = Todo.new(params[:todo])
	if @todo.save
		location_string = @todo.name.split('at').last.strip
		@todo.tags.create(name: "location:#{location_string}")
		redirect_to root_path
	else
		render :new
	end
end

it "creates two tags with two locations" do
	post :create, todo: {name: "cook at home and work"}
	expect(Tag.all.map{&:name).to eq('location:home', 'location:work')
end

def create
	@todo = Todo.new(params[:todo])
	if @todo.save
		location_string = @todo.name.split('at').last.strip
		locations = location_string.split('and').map(&:strip)
		locations.each do |location|
			@todo.tags.create(name: "location:#{location}")
		end
		redirect_to root_path
	else
		render :new
	end
end

it "creates multiple tags with four locations" do
	post :create, todo: {name: "cook at home, work, school and camp"}
	expect(Tag.all.map{&:name).to eq('location:home', 'location:work', 'location:school', 'location:camp')
end

def create
	@todo = Todo.new(params[:todo])
	if @todo.save
		location_string = @todo.name.split('at').last.strip
		locations = location_string.split(/\,|and/).map(&:strip)
		locations.each do |location|
			@todo.tags.create(name: "location:#{location}")
		end
		redirect_to root_path
	else
		render :new
	end
end

Video #2
Interactive debugging for solution discovery 
Sanity test after finish test - make sure things work in browser
Oops, creates a tag with a location with no location!
Rather than diving into code, write test case that goes after bug

it "does not create tags without inline locations" do
	post :create, todo: {name: "cook"}
	Tag.count.should == 0
end

want to run debugger in context of one test

rspec spec/controllers/todos_controller_spec.rb:53

location_string 
"cook"

oops! 
in an irb session.

$"abcd".split("at")
=>"abcd"

There's your problem. How do we fix this?
"cook".slice(/.*at(.*)/, 1)

now error due to nil
.try(:strip)

introduced with 1.9.

if location_string
	locations = etc.
end

now run all rspec - all-green test suite

Video 3 
Responding to feature changes with tests
bug "eat an apple"
tag
location: an apple

it "does not create tags with at in it without inline location"
post :create, todo: {name: "eat an apple"}
expect(Tag.count).to eq(0)

to fix, make sure at is a word
in regex
.*\bat\b(.*)

b's are boundaries

works!
but what if: 
"get good at swimming"

flaw in design
not really a bug, but bad design
didn't really think that "at" might not indicate location
need a new direction for product

decide to use upcase AT instead
get good at swimming
cook AT home

implement new spec first

it "creates a location tag with upcase AT" do
	post :create, todo: {name: "shop AT the Apple Store"}
	Tag.all.map(&name).should == ["location: the Apple Store"]
end

going to break other tests, but that's ok
get it to work first

regex
\bAT\b

now just run specific spec
passes

run rspec overall
fixed tests to use "AT" rather than "at"
all pass
pretty simple fix
sometimes when product direction changes, may not be this easy
key to have all the tests in place!

Video 4
Transactions

We want to verify that list orders can only be integers
Doing a batch update, can't just check one queue item at a time in form

Go read ActiveRecord transaction
They all succeed, or they all fail

HW
Change order of items in the queue
Should be ordered from 1-4
User can change numbers in field
User input should always be integer, validate
If change a number to 1 more than queue length, video moves to end
go look at transactions

Scratchpad/brainstorming
queue_item, validate position is_integer

queue_items_controller
update action

binding.pry - how are new positions passed in? array?
# sort all queue items by current position
# set queue_item.new_position = new positions
QueueItem.each.transaction do |queue_item|
	if position > current_user.queue_items.length
		queue_item.new_position = current_user.queue_items.length
	end
	queue_item.position = queue_item.new_position
	end
end

, name="#{queue_item.id}"
, queue_item_path, method: :update,
update_queue_path(queue_item_positions[]), method: :update, 

yeahhh... several hours later, I watched the solution
 = hidden_field_tag "queue_items[][:id]", queue_item.id
 the first [] is telling the form that it's an array, adding another element
 the second one is the actual key.

I used a separate conditional wrapper for extra conditions, but here's how to use a transaction.
begin
	ActiveRecord::Base.transaction do
	  #stuff you want to revert if one of them fails. Use a bang to raise exception!
	end
rescue ActiveRecord::RecordInvalid
	flash[:danger] = "Invalid position numbers."
	# etc.
	return
end

refactor todos controller
be very careful about putting business logic in the controller
better solution: push down to model level

if @todo.save_with_tags
	redirect_to root_path
else
	render :new
def save_with_tags
	if save
		location_string = name.string
		etc.
		true
	else
		false
	end
end

Do you need a new test now? Still covered in controller.
If a very simple move refactor, not worth changing test.
However, if implementing new features, consider writing new tests on the model level.

Make private method, create_location_tags 

Skinny controller, fat model - move stuff down to the model if you can!
Fat controllers in the wild
redmine, chiliproject
redmine - look at index, show - very hard to know what's going on

run the whole test suite using "rspec" to make sure you didn't break anything!

HW - set rating
TDD pattern
If no review, appear blank
Note that review validation requires review - need to find out way to bypass validations

