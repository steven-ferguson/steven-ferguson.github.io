---
layout: post
title: "Test your API with RSpec"
date: 2014-09-18 21:03:11 -0700
comments: true
categories: [Ruby on Rails, RSpec, Testing]
keywords: "rails, api, test, ruby"
description: "How to test an API with RSpec Shared Examples"
---

I have been writing a lot of tests for our API lately, and I wanted to share a technique that has helped me build a test suite that is easy to understand and write. Our API keeps a pretty consistent pattern which makes it a perfect candidate for [RSpec shared examples](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples). Similar to  [RSpec shared context which I discussed in another post](http://steven-ferguson.github.io/blog/2014/05/17/using-rspec-shared-context-for-dryer-specs/), shared examples are a great way to clean up test code and create useful abstractions.

To include a shared example in a spec you have to use one of the following methods:
```ruby
include_examples "name"      # include the examples in the current context
it_behaves_like "name"       # include the examples in a nested context
it_should_behave_like "name" # include the examples in a nested context
matching metadata            # include the examples in the current context
```

These methods allow you to pass parameters that can be used in the example group. I utilize this for my api tests to provide a factory name, the path for the request and the model:

```ruby
# /spec/requests/account_spec.rb

require 'rails_helper'

describe 'Account API' do
  describe '/api/accounts' do
    it_behaves_like 'an index route', Account, :account, '/api/accounts'
  end
end
```

In a Rails project, it is usually recommended to include shared examples under the support folder, so that they are autoloaded and available to use in all your tests.

```ruby
#/spec/support/example_groups/index_route

shared_examples "an index route" do |model, factory, path|
  let(:resources) { model.to_s.tableize }

  it "responds with a collection of the resource", type: :request do
    resource = FactoryGirl.create(factory)

    get path

    json = JSON.parse(response.body)
    expect(response.status).to eq 200
    expect(json[resources].length).to eq 1
  end
end
```

Now when I want to test another API endpoint that follows the RESTful index pattern, I can reuse this spec. Another benefit is that if I were to change the behavior for the index action by requiring an oauth token, I would only need to make the change in one place. Shared examples are also composable, so you could create a shared example for a restful resource by combining shared examples for all the different route types:

```ruby
shared_examples "a RESTful resource" do |model, factory, path|
  it_behaves_like "an index route", model, factory, path
  it_behaves_like "a create route", model, path
  it_behaves_like "a show route", model, factory, path
  it_behaves_like "an update route", model, factory, path
  it_behaves_like "a destroy route", model, factory, path
end
``` 