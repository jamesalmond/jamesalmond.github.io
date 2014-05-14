---
title: Testing asynchronous Sinatra with RSpec and Cucumber
layout: post
---


I've been using some EventMachine and asynchronous Sinatra stuff recently and the change in paradigm to event-driven code makes testing a little more difficult. I tend to like to do integration tests using Cucumber but getting an app running within Cucumber was troublesome. Luckily the async_sintra library comes with some test helpers that integrate with rack-test built in that make testing a little easier.

``` ruby
ENV["RACK_ENV"] = 'test'
require 'rspec'
require 'test/unit'
require 'rack/test'
require "sinatra/async/test"

module AppRunner
  def app
    My::App.new
  end
end
World(Test::Unit::Assertions, Sinatra::Async::Test::Methods, AppRunner)

def post_json(path,json)
  header 'Content-Type', 'application/json'
  apost path, json.to_json
  em_async_continue
  last_response
end
```


The helpers have a few parts that rely on test/unit, which I dont use, so we have to include the test/unit assertions into the world. We can then use the aget, apost, aput etc. helpers (in a similar way to rack-test) to make requests to the app set up in the World block. The 'em_async_continue' call cranks up Eventmachine to ensure anything queued within Eventmachine gets completed before we try to access the body.

You can use a similar technique to test your apps in RSpec:

``` ruby

require "spec_helper"
require "sinatra/async/test"
require 'test/unit'
module My
  describe App do
    include Sinatra::Async::Test::Methods
    include Test::Unit::Assertions
    
    def app
      App
    end
    
    subject { apost "/some/url", import_json }

    it "should do something" do
      last_response.body.should ...
    end
    
  end
end
```
I'm not sure this is the best way to integrate async_sinatra into a test suite but after fruitless hours of trying other ways it seems to do the trick.

