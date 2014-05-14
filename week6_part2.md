Payment processing

blog - 37 Signals - "How do you process credit cards"
2007 - huge headache
particularly recurring subscription
Paypal - unprofessional - have to direct user to site, controls all payment data - can't analyze
Today's moving parts
Need to have a merchant account
Account with authorize.net - payment gateway - take info, process charge, deposit to merchant
Active Merchant - Ruby gem recurring charge - requires lots of custom code - scan every night,
submit again, if declined, try again, if declined again, then lock user account

Not mentioned - 
PCI compliant - Payment card industry data security standard
Have build and maintain secure network
Protect cardholder data
Maintain vulnerability management program
Monitor test networks
etc. - lots of things

If you're building a small app - will become too prohibitive
Stripe - pioneer makes it easy

Don't need merchant account, payment gateway
Huge relief! (Don't have to go to local bank, fill out paperwork, etc)
Stripe set up account, can go ahead and play
When ready, go ahead and link Stripe to bank account
Stripe.js - customer info never through own server, straight to Stripe
Price - 2.9%, plus 30c per charge - includes fee for credit card network
Stripe has really good support
Kevin's worked with them directly
Only available in US or Canada right now - working hard on rollout
But can accept credit card payments all over the world

How to create a simple charge with stripe
Sign up for stripe
Currently in test mode - transactions experimental, credit cards won't be really charge
(Flip to live mode - need to put in banking information)

gem 'stripe'
bundle

rails c
copy code from Stripe site showing API
use the test secret key
amount is in cents
stripe.com/docs/testing shows sample card numbers for different card types

Stripe sends back JSON (note "livemode": false)
Note that did not provide address, city, zip code, etc.
With Stripe you don't have to do it - only for information purpose (!!)
Refresh Stripe account page - see $4 charge

Accept payments with checkout
server needs to have the card information to send to stripe
go to references/checkout on Stripe site
can have a button that pops up a page

How to do with our todo app
Added a link on the front page

resources :payments, only: [:new]

%p Thank you for your support
= form_tag payments_path do
(script copied from Stripe goes here)
%script(rest of code)
Actually loading a JS checkout.js from stripe.com
data-key - publishable test key - stripe is smart enough to include this on the copiable code

That's all you need! 
But you don't have a create action in the payments controller

def create
	require 'pry'; binding.pry
end

check params
stripeToken - actual number never through server - bypass PCI compliance

check charging card info on page
copy into create (uh... shouldn't we use an environmental variable here to hide the secret test key?)

What happens... the popup hits the Stripe site, gets a token back, and then submits that to the server

def create
	Stripe.api_key = "testkeycopied"
	token = params[:stripeToken]
begin
	charge = (etc - note inclusion of token)
	flash[:success] = "Thank you for your generous support!"
	redirect_to new_payment_path
rescue Stripe::CardError => e
	flash[:danger] = e.message
	redirect_to new_payment_path

yep, payment came through already

stripe testing - card numbers that trigger errors - 4000000000000002 - decline failure

HW - charge customer's credit card when they sign up - skip form, charge 9.99 when they click a make payment and sign up Stripe button - stock form that shows up

Don't worry about testing, will cover in future

image broken

redeploy master

  def create
    @user = User.new(user_params)
    if @user.valid?
      Stripe.api_key = "sk_test_tO8yd15PGNgYGIUMU5Y5Vlgr"
      stripe_token = params[:stripeToken]
      begin
        charge = Stripe::Charge.create(
          :amount => 999, # amount in cents, again
          :currency => "usd",
          :card => stripe_token,
          :description => "#{@user.email}"
        )
        @user.save
        handle_invitation
        AppMailer.registration_email(@user).deliver
        flash[:success] = "You registered! Welcome, #{params[:user][:full_name]}!"
        redirect_to login_path
      rescue Stripe::CardError => e
         flash[:danger] = e.message
         render :new
      end
    else
      render :new
    end
  end

use Figaro to manage Stripe keys
creates application.yml file
in code, Figaro looks at environment.

gem 'figaro'
rails g figaro:install

set keys in application.yml
# config/initializers/figaro.rb
Figaro.require("pusher_app_id", "pusher_key", "pusher_secret")

figaro heroku:set -e production


Accepting payments with custom form integrated into interface
= form_tag payments_path, id: 'payment-form' do

include script tag as part of page	
content_for :head do
	%script(src="https://js.stripe.come/v1/")
	:javascript
		Stripe.setPublishableKey(ENV['STRIPE_PUBLIC_KEY'])

= javascript_include_tag 'payments'

HW - include credit card fields on registration form
Charge $9.99 when they hit button
Only create record if successful
Don't worry about testing.
