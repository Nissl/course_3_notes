Background jobs

In the todos create action, Appmailer is called.
However, email sending isn't very fast.
Relies on third party application, typically.
Before email sent, controller doesn't generate response to user.
To the user, this is going to feel slow.
Other requests to the server also get queued. 
Email sending doesn't need to be synchronous - can be several seconds, even a few minutes later.
Offload email sending to something called a background process.

Background process by comparison with web/foreground process.
Doesn't interfere with web process.
Now predominant architecture for Rails:

requests -- >         web processes
                      web processes

background jobs -->   background worker processes
					  background worker processes

Each web process (JM: heroku dyno?) has an instance of Rails
Something that takes a while and is not time sensitive gets offloaded
Think of it as a queue
Can have 1 or many background workers

Popular background jobs processing frameworks
There are several - Resque, Sidekiq will be focused on
Resque - out of Github - read intro post
Resque, Redis - watch on Railscasts
Redis is an in-memory key-value store that Resque relies on.
Sidekiq is a high-performance option that is compatible with Resque
Watch Sidekiq Railscast, episode 366

Resque Github intro post
DelayedJob improved - good enough if your site is not 50% bg work like Github
Need:

Persistence
See what's pending
Modify pending jobs in-place
Tags
Priorities
Fast pushing and popping
See what workers are doing
See what workers have done
See failed jobs
Kill fat workers
Kill stale workers
Kill workers that are running too long
Keep Rails loaded / persistent workers
Distributed workers (run them on multiple machines)
Workers can watch multiple (or all) tags
Don't retry failed jobs
Don't "release" failed jobs

Redis:
Atomic, O(1) list push and pop
Ability to paginate over lists without mutating them
Queryable keyspace, high visibility
Fast
Easy to install - no dependencies
Reliable Ruby client library
Store arbitrary strings
Support for integer counters
Persistent
Master-slave replication
Network aware

Railscast: Resque
Code-sharing snippet site
Syntax highlighting from external webservice
Want to handle that with a background job

def create
	@snippet = Snippent.new(params[:snippet])
	if @snippet.save
		url = URI.parse('http://pygments.appspot.com')
		request = Net::HTTP.post_form(uri, {'lang' => @snippet.language, 'code' => @snippet.plain_code})
		@snippet.update_attribute(:highlighted_code, request.body)
		redirect_to @snippet, :notice => "Successfully created snippet."
	else
		render 'new'
	end
end

Uses Pygments service - Trevor Turk - neat! Avoid local dependency.
But might be slow, don't want to tie up Rails process.
Redis - go later

brew install redis
run redis server command to start

gem 'resque'
bundle install

/lib/tasks
resque.rake

require 'resque/tasks'
task "resque:setup" => :environment #loads Rails environment when workers startup
# however, if you want to keep your workers light, may need to do something custom (not covered)

$rake resque:work QUEUE='*'
script does not output anything, but is working

now, add job to queue
# need to access snippet model
Resque.enqueue(SnippetHighlighter, @snippet)
# but this is converted to JSON, stored in Redis database.
# so better to pass in something like @snippet.id

app/workers
snippet_highligher.rb

class SnippetHighlighter
	@queue = :snippets_queue
	def self.perform(snippet_id)
		snippet = Snippet.find(snippet_id)
		url = URI.parse('http://pygments.appspot.com')
		request = Net::HTTP.post_form(uri, {'lang' => snippet.language, 'code' => snippet.plain_code})
		snippet.update_attribute(:highlighted_code, request.body)
	end
end

Web interface - Sinatra app
resque-web
one failed job
can check out why
restart rake task now that SnippetHighlighter created
cool.

How can you embed the admin panel into Rails app?

routes.rb
mount Resque::Server :at => "/resque"

gemfile:
gem 'resque', :require => 'resque/server'

now can just go to /resque/overview
but want to add some authentication so not public

could put it into an 
authenticate :admin do
end
block

config/initializers
resque_auth.rb

Resque::Server.use(Rack::Auth::Basic) do |user, password|
	password == "secret"
end

Move password == "secret" into external file not on repo
How to choose between this and others?
Web interface is really nice.
Github uses Resque to handle heavy loads - well tested, supported
If Redis dependency is a problem, Delayed Job might be right
But it loads the Rails environment for all workers
Both do polling, causes slight delay
If, say, doing online game that requires quick response, look at Beanstalkd.

Sidekiq
This gem is most similar to Resque
Handles multiple jobs concurrently using threads instead of processes, save on memory.
As with Resque...

def create
	@snippet = Snippent.new(params[:snippet])
	if @snippet.save
		url = URI.parse('http://pygments.appspot.com')
		request = Net::HTTP.post_form(uri, {'lang' => @snippet.language, 'code' => @snippet.plain_code})
		@snippet.update_attribute(:highlighted_code, request.body)
		redirect_to @snippet, :notice => "Successfully created snippet."
	else
		render 'new'
	end
end

Another good reason to move external services into background - if down, user not affected directly.
Uses Redis, like Resque
(installed Redis)
gem 'sidekiq'
app/workers
pygments_worker.rb

def create
	@snippet = Snippent.new(params[:snippet])
	if @snippet.save
		PygmentsWorker.perform_async(@snippet.id)
		redirect_to @snippet, :notice => "Successfully created snippet."
	else
		render 'new'
	end
end

class PygmentsWorker
	include Sidekiq::Worker

	def perform(snippet_id)
		snippet = Snippet.find(snippet_id)
		url = URI.parse('http://pygments.appspot.com')
		request = Net::HTTP.post_form(uri, {'lang' => snippet.language, 'code' => snippet.plain_code})
		snippet.update_attribute(:highlighted_code, request.body)
	end
end

$sidekiq
$bundle exec sidekiq

Stuff to keep in mind: It will keep retrying a job if there's an error
Look out for side effects! (resending email, very bad!)

#disable retries
class PygmentsWorker
	include Sidekiq::Worker
	sidekiq_options retry: false
end

All code used by a worker should be thread safe
Don't share data that's mutable between instances - instance variables
All libraries your worker uses need to be threadsafe
Be aware of pool limit - how many threads can connect to database at same time
May want to bump this up... as high as 25, which is default limit for sidekiq

Can schedule time:
PygmentsWorker.perform_in(1.hour, @snippet.id) # good for clearing caches, though nonsensical here

Prioritize queues:
sidekiq_options queue: "high"
#if don't specify, will be default

$bundle exec sidekiq -q high,5 default
now the high queue takes priority

deployment - Capistrano recipe include in config/deploy.rb
pass in settings via sidekiq.yml file in config

worker monitoring - web interface like Resque
require 'sidekiq/web'

Example::Application.routes.draw do
	mount Sidekiq::Web, at '/sidekiq'
end

gem 'sinatra', require: false
gem 'slim'

lots of cool stuff!
look at sidekiq wiki for password protection method

Let's check out Worker from Sidekiq::Worker
sidekiq/lib/sidekiq/worker.rb

Check out middleware
client-side and server-side
Handles out logging, retrying, exception handling

Might want to consider writing your own!

processor - handles job after comes in from redis 
invokes middleware, calls worker

Key to multithread, Celluloid - great way to deal with it in Ruby.

Tealeaf walkthrough of Sidekiq
Sidekiq is a gem
app/workers
include Sidekiq::Worker
define perform method
HardWorker.perform_async

There is a wiki that goes to Sidekiq project
Install and configure with Redis

Best practices
- make sure you do not pass in objects
- make workers idempotent and transactional
idempotent: retry friendly
- don't refund the customer multiple times!
- check if refund already processed

Delayed extensions
- convenient setup for ActionMailer (and ActiveRecord)

Testing
Sidekiq pushes jobs into an array - treat the workers themselves like Ruby objects

Sidekiq in action
Todos app

have sidekiq installed

AppMailer.delay.notify_on_todo()
testing workers inline

spec_helper:
require 'sidekiq/testing/inline'
Note: API has updated, see:
https://github.com/mperham/sidekiq/wiki/Testing

HW
Use sidekiq to turn invite email into a background job
Install redis (download) and sidekiq (gem)
Make sure you go through this locally and it gets sent
Make sure your tests still pass 

Procfile and Foreman
you have both web and background processes - start rails and sidekiq

web: bundle exec rails server -p $PORT
worker: bundle exec rake jobs:work (not for Sidekiq)
worker: bundle exec sidekiq

if push to heroku, will setup processes

go a step farther, foreman start locally
good because local load 

heroku ps - all processes currently running
heroku logs - web and worker process
heroku comes with a free web process, but not free with more.

Unicorn
alternative rails server
problem solved: rails handling one request at a time

gem 'unicorn'
copy config file on Heroku to config/unicorn.rb

in that file:
worker_processes - how many rails processes
either ENV["WEB CONCURRENCY"] || 3
Heroku good number

read config file explanation

web: bundle exec unicorn -p $PORT -c .config/unicorn.rb

HW
use unicorn as web server on heroku
install redis to go

check coderwall - free background jobs via hack - don't use in real production!
heroku config:set REDIS_PROVIDER=REDISTOGO_URL

staging
heroku fork -a sourceapp targetapp (tl-myflix, tl-myflix-stage)
git remote add staging git@heroku.com:tl-myflix-stage.git
git push staging mod8:master

Various ways to copy over database from production to staging from time to time (follower costs $$)
But staging server sends emails, so be careful! Add line to send email to admins instead, perhaps.

Deployment pipeline
Many ways to do this. We'll use a simple one.

1. From now on, merge feature back into master before deploying anywhere
2. Run Rspec again, make sure nothing got broken.
3. Deploy code to staging environment, manually test on staging server
4. Deploy code to production server.

Setup Sentry via Heroku
gem 'sentry-raven'

You can now see runtime errors in the Heroku console. Cool beans.

Paratrooper gem
automate deploy pipeline
put example usage code in lib/tasks as deploy.rake (w/tweak for Heroku app names)
you can now run rake deploy:staging and rake deploy:production
contains match-tag line to staging. cannot skip staging step now.
note to self: only deploys off of master!

Once your git flow is approved,
merge it, pull, and
deploy to staging and production.
