Integration testing with emails
Doing stuff directly with Actionmailer is kinda low-level
Capybara-email gem
add
gem 'capybara-email'

clear_emails
visit email_trigger_path
open_email('email_address_here')
current_email.click_link

Nice gem!

HW
write integration test for resetting password
user sign in
click on forgot password page
enter email
send email
open link
user click link in email
user puts in new password, clicks button
sign in with new password on sign in page

HW
Invite a friend
Friend's name, email, message
Send email to friend
Go to register page
Email address is prefilled
Application automatically connects to inviter after follow
Do feature, controller, model level tests

double logout in different windows results in looking at code

Concerns
ActiveSupport::Concerns
Create token before todo - what if multiple models with token?
Don't Repeat Yourself!
Formally - every piece of knowledge should have one authoritative definition in a software application.
Extract and take to one place. 

lib/tokenable.rb

module Tokenable
	extend ActiveSupport::Concern
	included do
		before_create :generate_token
	end

	private

	def generate_token
	 self.token = SecureRandom.urlsafe_base64
	end
end

class Todo
	include Tokenable

but in Rails 3, lib directory no longer loads automatically!

require_relative '../../lib/tokenable'

could also move it to app/models folder - would load automatically

could also go to config/application.rb and 
config.autoload_paths << '#{Rails.root}/lib'

HW
Refactor
Find models that do token generating
Extract into concern
Instead of testing token generating
Create shared_example all tests refer to

HW
reading assignment
DHH (Rails creator)
"Put chubby models on a diet with concerns"
There is a debate in the community - some devs don't like this organization
Not truly independent of the models
Kevin: make sure tokenable is truly orthogonal to the model - both when and how independent. Do if/else with the type of model referrring to it.

Email service providers
Before, we were sending emails from gmail account
There's a clear daily sending limit of 2000/day
Sending emails should be offloaded to 3rd party vendors
Kevin recommends Mailgun and Postmark
Go read the Quora talking about why they're pricey
Opt for deliverability, not just sending.
In this course, we'll use Mailgun.

Mailgun
Email for developers
Powerful, low-level APIs
Track everything - click, open, unsub, bounce, complaint
Heroku mailgun free plan

heroku addons:add mailgun:starter

Actionmailer::Base.smtp_settings = {
	(MAILGUN SETTINGS FROM HEROKU HERE)
}

Make sure to leave delivery method as smtp!
We can use smtp, because Rails app_mailer prepares in proper format.
But you could also send HTTP - HTTP post - all info in the post.
Gives you a lot of flexibility!

HW
Use Mailgun w/SMTP mode to replace gmail in your application