Today's topic is `When APIs Change`. We're going to go over some background on the code. We're going to work on identifying the problem. We're going to look at caching requests with VCR, then we're going to fix the outdated tests. We'll talk about fast tests versus slow tests. And, finally, we'll have some conclusions and tips.

## Put it into context

We're going to be looking at an example app I wrote in response to an old peepcode screencast episode. The job of this gem is to go out to githubs API, and pull back some `json` information of a specific user's events. 

So, essentially anytime as a user of github you push to a repository or you fork a repository, or you leave a comment, or you create a pull request. There is a score associated with that in this app. Github just returns the event that you did, and this app's job is to score it. So in the events folder inside of the fantasyhub folder there is essentially one file for every event. And, if we look at, say, pull request ... 

![fantasyhub folder](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/001)

To submit a pull request is worth five points. This object was implemented as a module that `extends self`, so you don't ever create instances of a pull request event. It's just a type, it's a value object, it's very similar to the number forty-two. `Extend self` in this context is the same as putting `self` here. This allows us to call this object, like this ... 

~~~ruby
require 'fantasyhub/events'

module Fantasyhub::Events::PullRequestEvent
  extend self

  def score
    5
  end

end
~~~

And the method is `score`, and so you pass the method score to the pull request event. You'll notice here that I don't create a new instance of the pull request event to score it. This could have been done other ways. I choose to do a self-extending module. Essentially it has a single method, it's called score. In the case of this particular event the score is five. If I was to open up, say, a create event, we look at this one. 

~~~ruby
# fantasyhub/lib/fantasyhub/events/create_event.rb
def score
    3
end

# fantasyhub/lib/fantasyhub/events/push_event.rb
def score
    7
end    
~~~

The score is three. A push event, the score is seven, and so on for each of these events. So that is how we map a `json` event to a specific score. Real quickly, every object in here has one job. An event is the representation of a `json` event. It just has an added reader to expose some attributes. Those attributes are implemented as instance variables inside of a class. The adders are passed in as a hash, and then we fetch each of the keys off of the hash in order to instantiate the object. 

~~~ruby
class Fantasyhub::Event

  attr_reader :actor, :event_type, :repo_url, :score, :created_at

  def initialize(hash)
    @actor      =  hash.fetch(:actor)
    @event_type =  hash.fetch(:event_type)
    @repo_url   =  hash.fetch(:repo_url)
    @score      =  hash.fetch(:score)
    @created_at =  hash.fetch(:created_at)
  end
end
~~~

I could have used open-struct, or a struct, or many other ways, but I like simple things, so I chose to use a class here. Because I'm using the fetch on the hash, if one of these keys are missing there will be an error and I will be alerted to exactly what key was missing, rather than getting some weird no-value. Inside of my feed folder, since I'm dealing with a feed of events in `json` format from github, I have three objects in here. One is just to download that feed. The next one is to parse it into something that I can understand in my app, and then the third part would be to then score the feed. 

![feed folder](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/002)

So, we also have a feed object, which again this is just a name space representing a feed, and that's because we have this folder here called `feed.` Same thing for events. Events is a name space, fantasyhub events, and we want to load all of the files inside of `event` that end in the post-fix of _event.rb. If github was to add a new event tomorrow, I could just create a new definition here, give it a score, and it would just work.

~~~ruby
#fantasyhub/lib/fantasyhub/events.rb
require 'fantasyhub'

module Fantasyhub::Events;end

Gem.find_files("fantasyhub/events/*_event.rb").each { |path| require path }
~~~
## Finding the problem

If I run my test runner, the tests are going to return in about a quarter of a second. There's twenty-eight of them with twenty-eight assertions and everything passes, it looks green and happy. But, if I prefix this with `slow`, another test will now run on top of those twenty-eight, which is an end-to-end integration test. This test fails because the github API has moved. To fix this, I first need to identify why my integration test is failing, yet, my unit tests are passing. And the answer to this is actually pretty simple. 

It's happening because I'm using this gem VCR, which allows me to record a conversation with a remote server into a cassette, which I can then run my tests against later, so that if I'm on a train, for instance, and I have no internet, the external connection doesn't matter because I can test against the cassette, which is just a response of the server when it was live. So, these cassettes are essentially a year old and they have data for the old API with the old API data. And, if I wanted to make my unit tests stop lying to me I can just do a `RM` test, fixtures, cassettes, because this is where all of my cassettes are stored. 

If I then again run my unit test, you can see a pause where the new cassette is being recorded, and two failures happened because now they, too, are getting this Open URI HTTP error, four ten gone zone. So, that's because the URL for github used to look like this: `github.com/userid.json`. That API URL now looks like this: `api.github.com/users/userid/events`. 

![OpenURI error](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/002/images/003)

So, let's look into this project again. Let's find where it's failing. It's failing in the downloader because it's the downloader's job to go out to github and get it. So downloader is just another self extending module. It has a single public method called `download`. You give it a UID and it just simply returns a feed in `json` format on that ID. So, the private method feed URL is wrong, it's pointing to the old value. We can fix that ... 

~~~ruby
# fantasyhub/lib/fantasyhub/feed/downloader.rb
def download(uid)
  feed_for(uid)
end
~~~

We're going to have to put `users` here, the UID and then we have to post-fix it with `events`. So now we have `api.github.com/users/#{uid}/events`. That will work. So, if I remove the cassettes that were recorded with the four ten error, and then I rerun my test suite for my unit tests, we can see that one of those tests that was failing before, passed. It doesn't get the four ten error. 

However, the value of the commits has changed, so the score has changed since the last cassette was recorded. So, I now have to go into the test fantasy hub, test, and change the score to match what the score looks like today. And the score looks one thirty-seven today. 

~~~ruby
require 'minitest_helper'

describe Fantasyhub do
  subject { Fantasyhub }
  describe "event_scores_sum_for(uid)" do
    it "must get the sum of the event scores" do
      VCR.use_cassette("tenderlove_activity") do
        subject.event_scores_sum_for('tenderlove').must_equal 148
      end
    end
  end
end
~~~

So, let's go to change this to one thirty-seven, and now I can rerun my tests, and I get twenty-eight out of twenty-eight passing units tests again, which is where we started. But, this time they're actually correct and if I throw in my `SLOW` flag to run my single end-to-end integration test, we can see that it is now fixed on the integration test as well as the unit tests.

Now I want to talk a little bit about fast tests and slow tests. It's important in your TDD cycle, if you're going to do TDD, and I highly recommend it, that your tests respond almost immediately. You can't have a bunch of set up time and a bunch of wait time when you're in the middle of a TDD cycle. If you lose the context, you lose everything. 

So, when you're in your TDD cycle it's beneficial to have all tests that are going to be systems tests, integration tests, whatever you want to call it, tests that test more than one thing and test more than one system. Essentially what I call slow tests, you need a way of segregating those. So, you might have noticed that I don't run my tests with rake, I run a test runner that I wrote. So if I was to open that up ... 

~~~ruby
#!/usr/bin/env ruby
exit(1) if __FILE__ != $0

require 'bundler/setup'
require 'webmock'

if ENV.has_key?("SLOW")
  if ENV.has_key?("CODECLIMATE_REPO_TOKEN")
    require "codeclimate-test-reporter"

    WebMock.disable_net_connect!(:allow => "codeclimate.com")
    CodeClimate::TestReporter.start
  end
end

$LOAD_PATH.unshift('lib', 'test')

fast_tests = `find ./test -name *_test.rb -print | xargs grep -l "minitest_helper"`
Dir.glob(fast_tests.split("\n")) { |f| puts f; require f }

exit(0) unless ENV.keys.include?("SLOW")

slow_tests = `find ./test -name *_test.rb -print | xargs grep -L "minitest_helper"`
Dir.glob(slow_tests.split("\n")) { |f| puts f; require f }
~~~

This is just a simple Ruby file, my test runner. We're going to look for an environmental key called slow. If that exists, we're going to look for the code climate repo token. If that exists we're going to tell WebMock to allow the code climate call to happen and we're going to make the test reporters start for that. The next thing is that we have to load the `lib` and the `test` directories to our Ruby load path so that we can require something like `fantasyhub` rather than `lib fantasyhub`. 

Now you can see I have what I call fast tests. And essentially I just run a units command to find all of the test files in the test directory that end with the _test post-fix. I print those out and then I pipe them into `xargs`, which is kind of like a four-each object. It's going to grep over each one of those and it's going to look for the string `minitest helper`. 

If it has `minitest helper` in it, it is going to be considered a fast test, not an integration test. We then glob over all of those files and we put each one of them into a require, which actually is what's responsible for loading the file. And if you load a file in Ruby it runs this test run. 

We then have an exit unless the key has slow in it. If the keys have slow in it, we're going to then do a very similar thing with our slow test. The only difference is actually that we're going to do a flag of capital l on the grep, which is going to be an exclusionary rather than an inclusionary search. 

So, any test_test.rb file that does not have `minitest` helper in it is going to be considered a slow test. So, same thing, we glob over those slow tests. We print them out to the console, and we require [00:12:00] them. Now I'm going to open up my integration test. So, we start by loading thincloud test, which is a test helper from the gym thincloud test. And we are going to then load fantasyhub, which is essentially the main entry port to the application. 

This is minitest using the spec version, so we're doing a describe. We're saying "Hey, this is the end-to-end live test against github". In my test I like to have a single subject so that I don't so multiple searches on multiple things. Similarly to, I don't like multiple instance variables in my views. The subject is just `fantasyhub.event_scores_sum_for(fantasyhubber)`. 

This fantasyhubber user is simply a user that's done one actions so I know that their points aren't going to change and throw my tests off. Now we're going to turn off the VCR so that it allows the request through WebMock. We're going to say that the subject, which is of course going to evaluate to `fantasyhub.event_scores_sum_for(fantasyhubber)`, that it's score is going to be equal to one. 

Now let me split my screen and I'd load up a fast test. So, a fast test ... The only thing that it includes at the top for require is just minitest helper. Minitest helper does all of the rest of the work. That's how I know it's a fast test. Minitest helper's going to load up VCR, load up WebMock. Here's where I call the VCR cassette that I want to use. There's where I test against the cassette.

~~~ruby
require "thincloud/test"
require 'fantasyhub'

describe "End to end live test against github" do
  subject { Fantasyhub.event_scores_sum_for("fantasyhubber") }

  it "expects the score for fantasyhubber to be 1" do
    VCR.turned_off do
      subject.must_equal 1
    end
  end
end
~~~

## Conclusions and suggestions.

So, some conclusions and tips for this episode: Move slow tests out of you TDD loop, make sure you always have an end-to-end smoke test, use VCR to record conversations with remote APIs, build small focused objects that are easy to change, write plain old Ruby objects whenever possible, use continuous integration testing to catch problems.

If you liked today's episode and you want to support high quality Ruby screencast content like this in the future, you can become a patron of the show by subscribing for only fourteen dollars a month to Rubycast.io. Not only does this help me produce excellent screen casts in the future, but it also gets you twenty-five dollars off of my hourly rate at codementor.io. That's it for this week's episode of Rubycast.io, but I will see you again next Tuesday where we will continue to dig into the full Ruby development stack. 
