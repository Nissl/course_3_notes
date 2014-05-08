Take a step back, look at testing style in Rails

In this course
1. mockups -> view templates
2. controller tests
3. model tests (if controller has complex logic)
4. feature specs/integration tests

Here, the controller is driving the models
Integration tests are a layer on top of view templates
This process is called "meet in the middle"

Inside-out
1. model tests
2. controller 
3. integration

Outside-on
1. integration

Here, integration is more vertical integration, exercises controllers and models as well as view. Integration written in very business-level description. Think about business value you're delivering - challenge you to build controller, model around that.

BDD: behavior-driven development
ABDD: acceptance-driven development 
- common in consultancies
- powerful working with clients 

advantage, comes from business requirements, more natural, don't need to think too much, just worry about the next thing

disadvantage, requires you to be really proficient with Rails and testing, need to write high-level integration spec (Cucumber or Rspec)

Inside-out
Less of a hurdle to start
But less clear where you're driving towards
Used more in product-type shops, where it's more about incrementally adding small things to a more mature product.

Meet in the middle
Straightforward to start from mockups
Advantage of outside in, drive out implementation
Not as rigorous as outside in, but easier to start, lower level
You don't have to write as many integration specs
(Integration specs are the slowest out of all tests)

In this course, we're sticking with meet in the middle.
But you should know what the others are.

HMT and HABTM
Look at has-many, through 
Physician can have many patients, one patient can have many physicians
Join table: appointments

class Physician < ActiveRecord::Base
	has_many :appointments
	has_many :patients, :through => :appointments
end

class Appointment < ActiveRecord::Base
	belongs_to :physician
	belongs_to :patient
end

class Patient < ActiveRecord::Base
	has_many :appointments
	has_many :physicians, :through => :appointments
end	

can say physician.patients
can say patient.physicians

the other way is has and belongs to many

class Assembly < ActiveRecord::Base
	has_and_belongs_to_many :parts
end

class Part < ActiveRecord::Base
	has_and_belongs_to_many :assemblies
end

advantage, simpler - don't have to name join
but always use HMT 
less simple, but good to always name join table
logic has a place to live
e.g. manual link for assemblies_parts telling people how to put together
4/5 times, come back to relationship

Overview of social networking in MyFlix
click on user name in show video page
shows profile page
videos that joe has in queue, reviews
click follow to follow joe
people page, shows joe, #videos in queue, #followers, unfollow button

HW1
implement user profile page
details: number of videos in queue, number of reviews
yay!

Self referential associations

2.10 Self Joins 
refer to other records in the same table

class Employee < ActiveRecord::Base
	has_many :subordinates, class_name: "Employee", foreign_key => "manager_id"
	belongs_to :manager, class_name => "Employee"
end

can retrieve employee.subordinates and employee.manager

watch Railscast 163 for more!

Railscast 163
user list, add friends
self-referencing association
not dealing with two separate models like normal

What do you do at user level?
controller level UsersController?
make add_friend, remove_friend?

No! Not standard REST model
Notice friend in multiple methods - add and remove as well - create/destroy

Create a new controller, a Friendships controller with create and destroy (not friends! you're not creating or destroying friends!)

Two different techniques (HABTM - not much these days)
HMT. 

create a new model friendship
user_id: integer
friend_id: integer
create destroy 

(actions)

class Friendship < ActiveRecord::Base
	belongs_to :user
	belongs_to :friend, :class_name => "User"
end

class User < ActiveRecord::Base
	has_many :friendships
	has_many :friends, :through => :friendships
end

friendships_path(:friend_id => user.id)

@friendship = current_user.friendships.build(:friend_id => params[:friend_id]

in show, how to access destroy?
just have @user

for friends in @user.friendships
	=friendship.friend.username
	= link_to "remove", friendship, :method => :delete

	@friendship = current_user.friendships.find(params[:id])

has_many :inverse_friendships, :class_name => "Friendship", :foreign_key => "friend_id"
has_many :inverse_friends, :through => :inverse_friendships, :source => :user

has_many :inverse_friends

HW2 
People page
Click name to user page
Show videos in queue
Show followers
Click unfollow to delete

Self-referential association

A little different from friending scenario, where association is multidirectional
Here, a follows b doesn't mean b follows a
Add some seed data, make sure to verify

HW
follow a person - makes you a follower of that person
(oops, already implemented that)

HW 
integration test
log in 
look at video
click on review
user profile page
click follow
click people
person should be in list
(not req'd, but should do: follow button disappears on their page)
unfollow
person no longer in list

