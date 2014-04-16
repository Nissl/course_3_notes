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
