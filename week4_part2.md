How to send emails

Todos app
Send email to current signed in user whenever you add one
Rails has an actionmailer class

app/mailers/app_mailer.rb

class AppMailer < ActionMailer::Base
  def notify_on_new_todo(user, todo)
    @todo = todo
    mail from: 'info@mytodoapp.com', to: user.email, subject: "You created a new todo!"
  end
end

mkdir app/views/app_mailer
create notify_on_new_todo.html.erb

<!DOCTYPE html>
<html>
<body>
  <p> You created a new todo! </p>
  <p> The todo is <%= @todo.name %>
</body>
</html>

controller

if @todo.save_with_tags
  AppMailer.notify_on_new_todo(current_user, @todo).deliver

check in log - yep, sent!

ActionMailer configuration

rails guides - action mailer basics (to read next)
use a lot:
SMTP settings
delivery method

config environments/development.rb - nope
config environments/production.rb - copy paste GMail SMTP
deploy to production, sends emails for real

development
not :smtp - just dumps in logs
gem
:letter_opener

group :development do
  gem 'letter_opener'
end

config.action_mailer.delivery_method = :letter_opener

now, in dev environment
presents email in browser

HW
read through action mailer basics on rails guides
default from:
set headers, attachments (know that they're there)
send email to multiple recipients by passing in array
using layouts
generating url_for (tricky - need to specify host)
config.action_mailer.default_url_options (host: "domain name")

quite a bit... whew!

Handling sensitive data
We talked about how to set up a gmail account to mail stuff out.
Obviously not good... can see username and password in full view, for anybody who can look at your source code.

Put them as environment variables on server
Go to heroku
Configuration and configvars - very easy to set in command line!

heroku config:set gmail_username=myflixemail.gmail.com 
(etc.)

:user_name => ENV['gmail_username']
:password => ENV['gmail_password']

railsapps github - article on environment variables
uses the same example as we gave
option 2 use Figaro gem
option 3 local_env.yml file, setting .gitignore

Heroku - persistent, manifested to application across deploys and restarts - only set them once

HW
Add sending a welcome email to the register process
Just say "welcome to myflix, dude!"
Deploy app to heroku, verify that email is actually being sent

Testing email sending
Open up tests for controller where email is sent

context "email sending" do
  it "sends out the email" do
    post :create, Fabricate_attributes_for(etc.)
    ActionMailer::Base.deliveries.should_not be_empty
  end

  it "sends to the right recipient" do
    # adding emails to queue - can't roll them back as transactions
    ActionMailer::Base.deliveries.should_not be_empty
    message.to.should == [alice.email]
  end

  it "has the right content" do
    (etc)
    message = ActionMailer::Base.deliveries.last
    message.body.should include("#{content}")
  end
end

HW
add tests to registration email sending

Setting random tokens
todos id is part of url
gives out a lot of information!
can use a random token instead of an integer
youtube already does this, for example
rails g migration add_token_to_todos

def change
  add_column :todos, :token, :string
  # if have millions, should use SQL to do this
  Todo.all.each do |todo|
    todo.token = SecureRandom.urlsafe_base64
    todo.save
  end
end

$rake db:migrate db:test:prepare


# Rails helper looks at to_param
# By default, just ID
Todos model:

before_create :generate_token

def to_param
  token
end

private

def generate_token
  # Use Ruby module SecureRandom
  self.token = SecureRandom.urlsafe_base64
end

def show
  Todo.find_by_token(params[:id])
end

HW
Forgot password
Add link to sign in page that takes to forgot
Put in email
Click send email
Show confirm password reset page
System sends an email
Link
password_reset/(token)
Click link, goes to reset password page
Changes password, takes them back to sign in page
Message password reset
Token should only be valid before user resets password
