Admins for application
Typical to have different types of users
Called "actors"

How to add admin functions?
in index action
if current_user.admin?
	Todo.all
else
	@todos = current_user.todos

ok if that's the only place... but cumbersome if you need to do it a lot of places
could have separate method, admin_index

def admin_index
	@todos = Todo.all
	render index
end

better, separated out... but what if admins can do a lot of things?
controller is going to bloat, hard to reason about things
plus have to manage access control very carefully

routes.rb

namespace :admin do
	resources :todos, only: [:index, :destroy]
end

check rake routes
added /admin/todos admin/todos#index
added admin/todos#destroy

app/controllers/admin/todos_controller.rb

# has a module, admin, in front of todos controller
class Admin::TodosController < ApplicationController
	def index
		@todos = Todo.all
	end
end

rails g migration add_admin_to_users

def change
	add_column :users, :admin, :boolean
end

rake db:migrate

rails c
User.create(username: "admin", full_name: "Admin User", email: 'admin@example.com', admin: true)

class SessionController
	def create
	redirect_to current_user.admin? ? admin_todos_path : todos_path
	end

views/admin/todos/index.html.haml
(copy over the index.html.haml, getting rid of the submission form)

separation gives you a lot of flexibility to, e.g., add a dashboard

Securing access
right now, anyone can go to admin/todos

could just add a before filter to admin/todos_controller.rb

probably ok for a very simple app without many controllers
but easy to miss in a big application

extend Admin::TodosController < AdminsController

class AdminsController < ApplicationController
	before_filter

	def ensure_admin
		flash[:danger] = 'You do not have access to that area.'
		redirect_to root_path unless current_user.admin?
	end
end

class TodosController < AuthenticatedController
class AdminsController < AuthenticatedController

class AuthenticatedController < ApplicationController
	before_action
end

In a web app, very common to have compartmentalized activity available only to certain actors.

HW
have admin add video page, but only for admin
add a boolean column to user indicating admin
then secure page, only admins can see it

Amazon S3 (simple storage service)
Cloud based storage service 
Can be used for application and user data
If your site serves a lot of content, then this is a good choice for handling storage
Getting started is pretty easy. Create account, create bucket to store data
Consider a client tool (transmit, cyberduck) if you interact with S3 regularly.

Carrierwave file uploading
The go-to gem for file uploads
Add the gem, do bundle
Now, generate uploaders

class VideoUploader < CarrierWave::Uploader::Base
	#file on server
	storage :file
end

uploader = VideoUploader.new
uploader.store!(my_file)

store - permanent storage
cache - temporary - validation errors (later)

add_column :videos, :small_cover, :string

class Video < ActiveRecord::Base
	mount_uploader :small_cover, VideoUploader
end

store the stuff in the public directory in your rails project
can add different versions of the same file
RMagick (imagemagick) adapter - install local & server
process :resize_to_fit

call uploader.url

Easy to setup with Amazon s3, use fog gem
don't put access key in source code! - set environmental variables
rec. minimagick over rmagick - don't need to bother with installs
gem minimagick

Railscast 253 - carrierwave to upload files.
Make a site, artists upload paintings and display in gallery
Right now, just add a name.
One option - paperclip (episode 134)
Carrierwave is more flexible - Rack based, Rails, Sinatra
Supports many ORMs 
everything in a separate class - uploader

rails g uploader image

rails g migration add_image_to_paintings image:string

class Painting < ActiveRecord::Base
	mount_uploader :image, ImageUploader

form_for @painting , :html => {:multipart => true} do |f|
	f.file_field :image
end

display
	= image_tag painting.image_url.to_s

resize inside imageuploader class
process :scale

or include CarrierWave::RMagick
# then can call to process method
# don't forget to add gem "rmagick" - or in our case, minimagick

version :thumb do
	process resize_to_limit => [200, 200]
end

= image_tag painting.image_url(:thumb)

/adding URL-based file
/name matters to carrierwave
= f.label :remote_image_url, "or image URL"
= f.text_field :remote_image_url

Carrierwave deals with getting the image, etc. automatically. Cool!
you can add a hidden_field such that it caches the file on a failed save

checkbox for removing files
S3 storage support (etc.)

HW
add video
pick large cover and small cover locally and upload them
send to S3 for storage

# note for tomorrow. So far, local upload works for both. No tests. Uh, how to test? Need to reupload all videos (how to seed? not sure of data structure, still. Also, getting rid of columns) 
# have tested the post create... not sure how to test uploader functionality on video, however.e
# fog working (!)
# how to seed? not sure
# how to test not sure

# whoops, missed setting up mini_magick, wasn't actually needed for this functionality
# also had to use brew to install imagemagick - note that it is part of the Heroku platform by default.

# go ahead and remove small_cover_url and large_cover_url columns
# dangerous operation if you have production data already!
# make sure you migrate the data before you remove columns, if you're already in production!
# Kevin not worried about seed data for now
# (Also not worried about making caching work for now) - not like I need to upload for my site, but still...

HW
add field to admin page allowing them to input URL of video (typically they don't upload here, will upload directly to S3 through a client)
when you hit watch now on the video page, the video should play

Note that Vmail, Wistia leading video platforms if you want to do something with it on your site
No TDD here - already did plubming - adding feature for user on page, will do feature test to verify.

HW 
Feature test for adding video
Login as admin
Add new video
Upload covers
Input URL for content
Sign out
Sign in as regular user
Browse to video
Check that cover there
Check that media file link there in browser