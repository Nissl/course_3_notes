You can do code first, test later. "Test later." Guard against regressions.

Code a little, test a little - smaller feedback loop - write code, then write tests, then write some more code, etc.

Test first development - Write tests, then write implementation code, make sure test fails, write implementation until passes
Benefit of test first development - must think up front what you want to do.

Test driven development - key word, driven. Test not only to set a goal/expected end result but also use tests to build up your implementation code, to "drive out" from the test. People tend to go for this when something is complex. 

Test driven design - on more of a system level - driving architecture

The above 5 categories are pretty fine grained, sometimes smushed together.

No one category is always superior. Depends on the problem. Pretty simple, test first development. Some experienced may write implementation code first. 

In this course: focus on code a little, test a little, and test first development, while touching on TD development.

Test-frst basic video

describe Todo do
  describe "#name_only?" do
  	it "returns true if the description is nil" do
  	  todo = Todo.new(name: "cook dinner")
  	  todo.name_only?.should be_true
  	  todo.name_only?
  	end
  	it "returns true if the description is an empty string"
      todo = Todo.new(name: "cook dinner", description: "")
      todo.name_only?.should be_true
  	it "returns false if the description is a non-empty string"
  	  todo = Todo.new(name: "cook dinner", description: "potatoes")
  	  todo.name_only?.should be_false
  end
 end

 class Todo < ActiveRecord::Base
   has_many :tags

   def name_only?
     true
   end
 end

 Only the last test forces generalization (triangulize?)

def name_only?
  description.nil? || description == ""  
  description.blank? # same thing, but better
end

"red green refactor cycle" - start with red, make green, then refactor while keeping green

Member routes and collection routes
Say we want to implement search for todos

todos/search?term="cook"
todos/:id/highlight

resources :todos, only: [:index] do
  collection do
    get 'search', to: 'todos#search'
  end

  member do
    post "highlight", to 'todos#highlight'
  end
end

HW1
implement search by title

class Video
  def self.search_by_title(search_term)
  end

Video.search_by_title("family")

return array of videos, or empty array if nothing
write tests first - no video, one video, multiple videos
look at search in ActiveRecord: LIKE

Ugh. one hour later, shouldn't have specified this for videos if wanted to search by date later
has_many :videos, -> { order "title" }
it { should have_many(:videos).order("title") }

For an instance method, use pound sign in tests.

Custom form builders

overwriting label
if errors, append span html element with class error
delegate control back to form builder
app/helpers/

class MyFormBuilder < ActionView::Helpers::FormBuilder
  def label(method, text = nil, options = {}, &block)
  	errors = object.errors[method.to_sym]
	if errors
	  text += " <span class=\"error\">#{errors.first}</span>"
	end
    super(method, text.html_safe, options, &block)
  end
end

= form_for @todo builder: MyFormBuilder do |f|
  = f.label :name, "Name"
  = f.text_field :name
  = f.label :description, "Description"
  = f.text_area :descritpion, rows: 6
  %br
  = f.submit "Add this todo"

Another way to do it:
1.
module ApplicationHelper
  def my_form_for(record, options = {}, &proc)
  	form_for(record, options.merge!({builder: MyFormBuilder}), &proc)
  end
end

2.
= my_form_for @todo do |f|

Form builders in the wild:
Formtastic: semantic_form_for
simple_form (also wrote Devise): support hints, inline labels, integrated with both Twitter bootstrap, zurb foundation 3. Both of those are popular styling frameworks

What Kevin didn't like: works well if your default style is similar to simple_form. But if it's not, spend a lot of time fighting. Default mapping, e.g. boolean, checkbox

bootstrap_form is where Kevin wound up. Still write text_field, password_field, etc. Little DSL with it, but automatically does e.g. alert_message. Keeps you from writing highly verbose markup for e.g. horizontal form.

HW:
hook up front page
hook up register page: email address, password, full name - individual error messages

user model: email_address, password, full_name

User.create(full_name: "Pete Sanchez", email_address:"psanchez@null.com", password: "petepass")

At the moment, the root/front page is done.
The user model is set up, with bcrypt password hashing.
The sign in and register pages load.

Need to wire up sign-in and register pages
(Incl. need to show custom errors)

class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
end
 
class Person < ActiveRecord::Base
  validates :email, presence: true, email: true
end

