Making a local wrapper for Stripe

May need to interface with Stripe in a lot of places
Can customize with account info, etc.

rails generate model stripe_wrapper

Does NOT need to extend from ActiveRecord::Base

other half of wrapping, handling responses

module StripeWrapper
	class Charge
		attr_reader :response, :status
		def initialize(response, status)
			@response = response
			@status = status
		end

		def self.create(options={})
			StripeWrapper.set_api_key
			begin
				response = Stripe::Charge.create(amount: options[:amount], currency:'usd', card: options[:card])
				new(response, :success)
			rescue Stripe::CardError => e
				new(e, :error)
		end

		def successful?
			status == :success
		end

		def error_message
			response.message
		end
	end

	def self.set_api_key
		Stripe.api_key = Rails.env.production? ? ENV['STRIPE_LIVE_API_KEY'] : "sk_test_etc"
	end
end

def create
	token = params[:stripeToken]
	charge = StripeWrapper::Charge.create(
		:amount => :3000,
		:card => token
		)
	if charge.successful?
		flash[:success] = "Thank you for your generous support!"
		redirect_to new_payment_path
	else
		flash[:error] = charge.error_message
		redirect_to new_payment_path
	end
end


Didn't follow a TDD process here.
Rspec models/stripe_wrapper_spec.rb
Can fake response data from third party API, but can also hit service
We're going to hit the service here.
Slow! Hits twice per test, to set the api key and then actually charge

describe StripeWrapper::Charge do
	before do
		StripeWrapper.set_api_key
	end

	let(:token) do
		Stripe::Token.create(
			:card => {
				:number => card_mumber,
				:exp_month => 3,
				:exp_year => 2016,
				:cvc => 314
				}
			).id
	end

	context "with valid card" do
		let(:card_number) { '4242424242424242' }
		
		it "charges the card successfully" do	
			response = StripeWrapper::Charge.create(amount: 300, card: token)
			response.should be_successful
		end
	end
	
	context "with invalid card" do
		let(:card_number) { '4000000000000002' }
		let(:response) { StripeWrapper::Charge.create(amount: 300, card: token) }
		
		it "does not charge the card successfully" do
			response.should_not be_successful 
		end

		it "contains an error message" do
			response.error_message.should == 'Your card was declined'
		end
	end
end

Right now 13s to run Stripe, 8s for specs, 5s for Rails load. 8s due to 2x server hit per test
So cannot afford to hit service every time
WebMock is a library that does exactly that.
But works on a low level, still have to do it manually
That is where VCR comes in - record & playback interactions in a data file
Install VCR and webmock (VCR supports others) in test group

spec_helper

require 'vcr'
(copy and paste in configuration code from VCR main page)

it "charges the card successfully", :vcr do

can see the files in spec/cassettes/StripeWrapper_Charge
Lots of options for customizing how you record interactions.
Can record :once
Can record :all - get new interaction by flipping this on and off

HW build a wrapper for Stripe, both implementation and test code

Differences from HW code, need to wrap body of each test in 
	VCR.use_cassette("cassette_name") do

	end

instead of just putting :vcr at the end of each line.

Namespace::class.method
in tests ".create" - instance method use pound sign, otherwise use . method

Test doubles and method stubs

tests for payments controller

describe PaymentsController do
	describe "POST create" do
		context "with a successful charge" do
			it "sets a flash success message"
			it "redirects to the new payment path"
		end

		context "with an error charge" do
			it "sets a flash error message"
			it "redirects to the new payment path"
		end
	end
end

Wrapper has been thoroughly tested, don't want to test again
Would clutter test with noise, take time
Want to stub method to return successful charge
test double = fake object
by doing this, immediately returns a test double 
if you call successful, it always returns true

charge = double('charge')
charge.stub(:successful?).and_return(true)
StripeWrapper::Charge.stub(:create).and_return(charge)

post :create, token: '123'
flash[:success].should == 'Thank you for your generous support!'

it "sets the flash success message"
	# (moved stubbing to before)
	expect(response).to redirect_to new_payment_path
end

context "with a failed charge" do
	before do
		charge = double('charge')
		charge.stub(:successful).and_return(false)
		charge.stub(:error_message).and_return("Your card was declined.")
		StripeWrapper::Charge.stub(:create).and_return(charge)
		post :create, token: '123'
	end

	it "sets the flash error message" do
		expect(flash[:error]).to eq("Your card was declined.")
	end

	it "redirects to the new payment path" do
		expect(response).to redirect_to new_payment_path
	end
end

HW
go read Rspec documentaion on test doubles, method stubs
then read all the utilities on ways to stub (substitute implementation)
then use the wrapper to charge the user's credit card when they sign up
if errors in field, don't charge credit card
if errors in payment, don't create user, put them back on page
only create user & charge card if payment valid and goes through stripe
(time to fix the users controller! yay!)

Feature tests for charging credit card
#in *theory* don't need to do all of these as wrapper and happy path tested but so important that worth testing.

require 'spec_helper'

feature 'Visitor makes payment', js: true do
	background { visit new_payment_path }

	scenario 'valid card number' do
		pay_with_credit_card("4242424242424242")
		expect(page).to have_content("Thank you for your generous support!")
	end

	scenario 'invalid card number' do
		pay_with_credit_card("4000000000000069")
		expect(page).to have_content("Your card's expiration data is incorrect.")
	end

	scenario 'declined card' do
		pay_with_credit_card("4000000000000002")
		expect(page).to have_content("Your card was declined.")
	end
end

def pay_with_credit_card(card_number)
	fill_in "Credit Card Number", with: card_number
	fill_in "Security Code", with: "123"
	find(#date-month).select("6-June")
	find(#date-year).select("2017")
	click_button "Submit Payment"
end

Always watch out for incompatibilities - upgrade Capybara and Firefox to fix
(Key is to upgrade selenium-webdriver)

Selenium is the slowest of the js runners despite being very visual, easy to debug.
Can also use capybara-webkit
Requires QT, install via homebrew

spec_helper:
Capybara.javascript_driver = :webkit

what if you want to run via selenium?

scenario "irrelevant", driver: :selenium do

Poltergeist is another nice headless runner (requires phantom.js install locally)
Transactions have been used to roll back the database after every spec
Works fine with reg test, but not with Selenium or webkit

Using database cleaner
Set config.use_transactional_fixtures = false

HW: write feature tests for registration process
test valid and invalid for top of form
test valid and invalid credit, as well as declined

only create user if valid

note to mention: solution may not handle invitation well - does it fail to load the invitation token again w/new page