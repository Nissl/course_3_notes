The base project is now Rails 4.0.1, but the solutions are 3.2.x

Big diffences:
1. Use strong_parameters in controllers
cmd-D or ctrl-cmd-G search word
double cmd-D select all

jQuery notes
modify h1 element
Javascript interact w/dom
Browser builds node model
Every browser has a slightly different DOM interpretation

jQuery(code);2. You need scopes to have a callable object. 


How can we find h1?
jQuery("h1");
$("h1");

How do we modify?
$("h1").text()

For example
Rails 3
has_many :videos, order: :name

Rails 4
has_many :videos, ->{ order(:name) }

Learn haml.
Do the tutorial on haml.info
Read through the reference.
(See haml_notes.md for brief notes on Haml syntax)

Prototype process for web apps
idea
wireframes
  low fidelity representation of application UI
  Balsamiq mockups
  purpose: communication with others & yourself
  easy to see what the workflow is, scope, main features
  don't overdesign the wireframe!
  do several versions - that's the point of wireframing
design
  engage a designer if you don't do it yourself
  layouts
  colors
  look & feel
  typically done in photoshop, give .psd files
slicing
  turn photoshop into mockups - several services you can hire if you don't have skills - turn into HTMl & CSS
  want to be pixel-perfect
development

Design to development handoff.
We're using a UI controller.
Can click on all of the different pages we're putting up - mockups - not backed by any data, pure images. Fake data

App/controllers/ui_controller.rb  

# check if environment, if so, redirect to root. don't show mockups to customers!

before_filter do 
  redirect_to :root if Rails.env.production
end

layout "application"

routes:
get 'ui/action', controller: 'ui'

views/ui:
By default, goes to index.html.haml
Displays links to all the other views (except index). 

app/assets/stylesheets:
application.css pulls in everything elsein the directory
Here, we're using one stylesheet per page. Other projects may merge into fewer files, or even keep everything in one file (really? yuk!)
We're using .sass files now. Better than vanilla CSS, easier to read and maintain.

There are two versions, sass and scss. Scss is a little more verbose, goal is to be fully compatible with CSS. Everything has curly braces and semicolons. sass-lang.com

We're using haml over erb. Uses indentation significantly. Don't have to close tags. It'll take a little while to learn, but not too difficult. 

Gemfile - haml-rails and sass-rails added. 
Read hashrocket blog post - design handoffs with the UI controller.

How to build from mockup to actual page:

want a /home directory to show videos. 
on home.html.haml view. 

on the view level:
create a videos directory
have index.html.haml and copy in home.html.haml
cut out extra categories.
@videos.each do |video|
  .video.span2
    = link_to "", video, nil do
      %img(src="#{video.cover_image_url}")

(Kevin is using vim :!mkdir appv/views/videos)
still need routes, controllers, but straightforward
Advantage of approach: once styles are setup, everything just works. Keep markups the way that they are. 

Seeding the development database with data records - quick population to play around with. Working with client, let the client play with app. 

controllers - just todo_controller

def index
  @todos = Todo.all
end

models - todo

db - one table, todo

views - todo - database
%ul
  - @todos.each do |todo|
    %li = todo.name

db/seeds.rb file
Todo.create(name: "cook dinner")
Todo.create(name: "eat")
Todo.create(name: "wash dishes")

$rake db:seed

HW1 create a video model
- title
- description
- small_cover_url - points to image on homepage
- large_cover_url - more like a poster - on actual video page

If you want to use existing covers, look at public/tmp

Create migration, model, put seed data in database

HW2 turn homepage into actual page
show all the videos from the database

HW3 turn video mockup into actual page
use tmp/monk_large.jpg for all images

HW4 build categories
migration, model, add category information to homepage

Sublime notes
cmd-T fuzzy search
cmd-D or ctrl-cmd-G search word
double cmd-D select all



