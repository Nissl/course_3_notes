Rspec macros

just added a before filter to the todoscontroller
tests are just hitting the controller directly - all fail

can obviously fabricate user, put them in the session
we have a huge number of specs, that would be a lot of repetition

put it into a before do for all of the specs, outside all of the describes

move it into a macro

mkdir spec/support
touch spec/support/macros.rb

def set_current_user
	john = Fabricate(:user)
	session[:user_id] = john.id
end

in main file

before { set_current_user }

def current_user
	User.find(session[:user_id])
end

Shared examples

spec: if user not signed in, redirect to sign-in page
we have a before that sets current user already, though!

add a macro:
clear_current_user 

def clear_current_user
	session[:user_id] = nil
end

need to do redirect in GET new, POST create, etc.

use shared examples
spec/support/shared_examples.rb

shared_examples "require_sign_in" do
	it "redirects to the front page" do
		response.should redirect_to root_path
	end
end

back in main rspec file:

context "not signed in" do
	before do 
		clear_current_user
		get :index
	end
	it_behaves_like "require_sign_in"
end

wait, that doesn't save any lines! would like to move the before do segment to the shared example as well, but the "get :index" line will be different for each.

pass in sandwich code using block

it_behaves_like "require_sign_in" do
	let(:action) {get :index}
end

it "redirects to the front page" do
	clear_current_user
	action
	response.should redirect_to root_path
end

HW: clean up your test code using macros and shared examples!

Feature specs
So far, writing specs for models (model specs) (unit?)
Controllers (controller specs) (integration?)
Views, routes, helpers, mailers - nope!
There are specs for each of these components in rspec
But we don't have to create an individual spec
Instead, people test integration of components in feature specs

Operate on a browser level - mimic user experience in the browser
Feature specs broken into features
Really a method of vertical integration

There's also horizontal integration across different elements
Called "request specs"

Feature specs - test from user browser, make sure feature implemented
Request - across multiple requests & responses, make sure thing happen in a sequence

Requires capybara gem
We're going to be using feature specs - we don't do request specs. If we want to test integration across multiple controllers, just write a feature spec
People do request specs to make sure a business process works. But then you might as well just use the feature spec, operating on the user level.

Capybara gem
Simulate how a real user interacts with browser

instead of before, background
instead of it, scenario
instead of let, given

Default driver for Capybara - RackTest 
very fast, does not fire up an actual browser. Does NOT support Javascript
If you have JS, need to work at Selenium or Capybara-webkit
Selenium fires up an actual browser - slow because of this
Capybara-webkit - install capybara-webkit - faster than Selenium, slower than RackTest

The DSL
(Huge amount of info - go look at documentation)
look up xpath

First feature spec

group :test do
	gem 'capybara'
end

bundle install

test helper file
spec_helper.rb
require 'capybara/rails'
put specs in spec/features
using feature/background/scenario style

create spec/features/user_signs_in_spec.rb

require 'spec_helper'

feature 'User signs in' do
	background do 
		User.create(username: "john", full_name: "John Doe")
	end

	scenario "with existing username" do
		visit root_path
		fill_in "Username", with "john"
		click_button "Sign in"
		page.should have_content "John Doe"
	end
end	

(connect "Username" to field via field_tag)
could use the name of the field or the ID
typical best practice: use the label text, easier to read

Watch railscast 257 on request specs and Capybara
This is a while ago, terminology out of date, general idea still applicaable

Railscast 257
(did do Cucumber)
will add tests to existing application, though likes TDD
call get tasks_path
response.status.should be(200)

rake spec
post_via_redirect - goes directly

using capybara alternative to webrat (not around anymore?)
added launchy gem as well as capybara

visit tasks_path
page.should have_content("paint fence")

visit tasks path
fill_in "New Task", :with => "mow lawn"
click_button "Add"
page.should have_content("Successfully added task")
page.should have_content("mow lawn")

launchy: call save_and_open_page - opens up browser at that point

Testing JS
<%= link_to_function "test js", '$(this).html("js works")' %>

it "supports js" do
	visit tasks_path
	click_link "test js"
	page.shluld have_content("js works")
end	

Fails! Capybara doesn't support JS, need to test via Selenium
point Capybara to latest and greates
gem 'capybara', :git => 'git://github.com/jnicklas/capybara.git'

in spec_helper
require 'capybara/rspec'

it "supports js", :js => true do

gotcha: database records
try it for "displays tasks"
database record isn't ready for tasks
change use_transactional_fixtures to false
however carries over database records between specs

gem 'database_cleaner'
bundle

spec_helper.rb
(copied from the database_cleaner documentation)
config.before(:suite) do
	DatabaseCleaner.strategy = :truncation
end

config.before(:each) do
	DatabaseCleaner.start
end

config.after(:each) do
	DatabaseCleaner.clean
end

HW: implement simple feature spec for sign in
come to sign in page, fill in email and password
click sign in 
verify that signed in 

HW: more integrated feature spec
log in 
go to videos page
go to specific video
press +my queue button
load my_queue, video should be added
follow link to video, go to video show page
video should be correct video, +my queue button should be gone
go to homepage, add a few more videos to queue
go to queue, reorder videos, update queue, come back in right order

test that queue related features work well together
tricky: videos have no anchor text, just images
how in capybara do you select the right video?

My solution to the last:
first("td input.form-control#queue_items__position[value='3']").set('1')
first("td input.form-control#queue_items__position[value='1']").set('3')

Other ways to do this stuff in homework
1. easiest
change ids - in view text_field tag id: "video_#{queue_item.video.id}"
fill_in "video_#{video1.id}", with: 3
expect(find("#video_#{video1.id}").value).to eq("3")

but sometimes, the front end relies on IDs

2. add data attribute (this is the final implementation I settled on)

data: {video.id}: queue_item.video.id
can't use with find, which only works for id or label

find("input[data-video-id='#{video1.id}']").set(3)
find("input[data-video-id='#{video2.id}']").set(2)
find("input[data-video-id='#{video3.id}']").set(1)

click_button "Update Instant Queue"
expect(find("input[data-video-id='#{video1.id}']").value).to eq '3'
expect(find("input[data-video-id='#{video2.id}']").value).to eq '2'
expect(find("input[data-video-id='#{video3.id}']").value).to eq '1'

3. look at table rows using scoping feature
within(:xpath, "//tr[contains(.,'#{monk.title}')]") do
	fill_in "queue_items[][position]", with: 3
end

expect(find(:xpath, "//tr[contains(.,'#{south_park.title}')]//input[@type='text']".value).to eq("1")

XPath rules
// starting from anywhere in html
//tr - anywhere tr
//tr[contains(., #{monk.title})] - the dot means anywhere in the current level
// again - anywhere under the tr
query using @

def expect_video_position(video, position)
	expect(find(:xpath, "//tr[contains(.,'#{video.title}')]//input[@type='text']".value).to eq(position.to_s)
end

def set_video_position(video, position)
	within(:xpath, "//tr[contains(.,'#{video.title}')]") do
		fill_in "queue_items[][position]", with: 3
	end
end