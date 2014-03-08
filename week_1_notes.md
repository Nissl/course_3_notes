Testing

Why do we test our software? 
(Automated tests)

Alternatives
- no tests at all (nobody does this... customers test)
- QA team - some large companies do this
	- developers work on code, throw it over to QA team
	- QA write test cases, go back to dev team with bugs
	- problems: 
		- organizational problems/mistrust - communication is often not great - different assumptions, different contexts
		- QA team gets blamed for lots of things, when works devs take credit, when doesn't work blame QA, plus QA puts pressure on dev team, dev team blams for slippage, etc.
		- often nobody wants to stay in QA, high churn results, entry-level, quality may be low
		- release cycle - two teams competing for time - delays

Impact of alternatives (especially no tests)
- code quality goes down, people don't care as much - devs get lazy/sloppy - as long as it appears to work, toss it along
- isolated knowledge: nobody else dares to touch an area that someone is working on, at extreme people may intentionally make code complex for job security
- how do you know things work, at all? - developer says, hey, works in a browser. Not good enough
- how do you know that things weren't broken when you added something? "regression problem"

Typical response: "I don't have enough time! I have a deadline!"

Technical debt

Different styles of development - TDD takes longer at the start/very small project, but as the size grows, it comes out way ahead. Without tests, takes longer and longer to add feature - complexity grows exponentially, and with it the likelihood that changes will break something. Then you have to go back, get all of your context back, etc. Fixes can break additional things, problems cascade all over project! The project can even become a rescue - hire from outside - those people generally build up a TDD pattern. Curve is also called technical debt.

Tests!
3 major types
- Unit
- Functional
- Integration

Unit test - test a component in isolation. Make sure all the cases are covered

Functional test - test multiple components in collaboration

Integration test - follow a business process, make sure components play well with other *and* achieve a business objective

In Rails:
Models, views, helpers, routes - unit tests
Controllers - functional tests - pull models, collaborate to generate data, views - one single request-response
Integration tests - emulate end user - drive through the browser - log in, click buttons, fill forms, submit - many request/responses

Multiple styles of testing within community, becoming more of an art - balance when, how much, what level you test.

This course will introduce one style, focus on unit tests and functional tests. We only write integration tests on important business workflows. 

Units: models, helpers
controllers
emulate user

cost/benefit analysis
unit tests close to code - write fast, test thoroughly
functional tests - in between
integration test - high level - lots of components - hard to test all the permutations & get good coverage, also slowest to run as you drive through the browser

speed: u > f > i
coverage: u > f > i
realistic: i > f > u 

1) Maximize, unit, functional tests
2) Integration tests for important business processes

First test in RSpec
Rspec is the testing framework in this course. Install Rspec, setup basic test

change gemfile, install to both test and development environments
bundle install
rails g rspec:install
creates .rspec file, spec directory, spec_helper.rb in spec

Let's check out the files.
.rspec --color # makes output of runs color coded
spec_helper.rb # configure rspec

$mkdir spec/models - all model tests here
todo_spec.rb:
require 'spec_helper' #rspec loads rails environment

describe Todo do 
end

run rspec from root directory
finished in a fraction of a second, green = pass - but didn't really test anything yet

rather than "tests", writing "specification" (R *spec*)

describe Todo do
  it "saves itself" do
  	todo = Todo.new(name: "cook dinner", description: "I love cooking!")
  	todo.save
  	Todo.first.name.should == "cook dinner"
  end
end

green dot on command line, passed

development vs. test database
check config, database.yml

db/test.sqlite3 - all tests segregated, clean slate
to bring up to date:
rake db:migrate db:test:prepare #forgetting this is a common error!

Github flow & code review in this course
- anything in the master branch is deployable
- to work on something new, create descriptively named feature branch off of master
git checkout -b branch_name
commit to that branch locally, regularly push work to same named branch on the server
git push origin branch_name
create pull request from your module branch to master branch
commits in review status - go ahead and review - review at least 3 others, once signed off

git checkout master
git pull origin master
deploy to heroku

HW1
install Rspec
write first test for video model - can save video

HW2, 3
validates_presence_of is also an option
model, order: is deprecated
#has_many :videos, order: :title #from examples, now deprecated
has_many :videos, -> { order "title" }
# need to learn more about scope blocks

# All 3 work, but last is preferred syntax now
#Video.first.title.should == "Test_Vid"
#ideo.first.should eq(video)

categories:
# different tests:
expect(Category.first.videos.length).to eq(2)
expect(comedies.videos).to include(futurama, south_park)

# my solution:
# expect(Video.first).not_to eq(video)

cutout pre-shoulda matchers:

it "belongs to category" do
	comedies = Category.create(name: "TV Comedies")
	video = Video.create(title: "Futurama", category: comedies)
	expect(video.category).to eq(comedies)
end

it "has many videos" do
	comedies = Category.create(name: "TV Comedies")
	# doing the same thing, multiple ways
	south_park = Video.create(title: "South Park", description: "super funny!", category: comedies)
	futurama = Video.create(title: "Futurama", description: "Fry is a cool dude", category: comedies)

	# only with order: :title in place in category model
	expect(comedies.videos).to eq([futurama, south_park])
end

  it "requires a title to save" do
    video = Video.create(description: "video with no title")
    expect(Video.count).to eq(0)
  end

  it "requires a description to save" do
    video = Video.create(title: "video with no description")
    expect(Video.count).to eq(0)
  end

# cut, testing Rails, not worth it (ditto category)
it "saves itself" do
	video = Video.new(title: "Test_Vid", description: "test_vid_description")
	video.save

	expect(Video.first).to eq(video)
end

Use shoulda matchers for testing basic model fatures

