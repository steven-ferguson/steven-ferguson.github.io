---
layout: post
title: "Using RSpec Shared Context for DRYer Specs"
date: 2014-05-16 16:30:35 -0700
comments: true
categories: [RSpec, Ruby on Rails, TDD]
---

Do you ever find that many of your specs have a similar setup that you are repeating over and over? I often find this happening when I'm writing feature specs that require a logged in user. For example:

```ruby    
feature 'some super cool feature' do
  background 'login an admin' do
    user = create(:user, role: 'admin')
    user.confirm!
    login_as(user, scope: :user)
  end

  scenario 'my super cool scenario' do
  ...
  end
end
```

If my login or authentication process changes, I have to manually go into each spec and change the code. I also find that this setup code clutters the spec and makes it harder to read. If we could extract the setup code and replace it with an easy to read single line, it would solve both of these problems. 

[RSpec shared contexts](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-context) provide just this type of solution. Using a shared context we can extract the setup into its own file, and rewrite our feature spec in a cleaner way:

```ruby
feature 'some super cool feature' do
  include_context 'a logged in admin'

  scenario 'my super cool scenario' do
  ...
  end
end
```

To create a shared context:
```ruby
# ./spec/support/shared_contexts/a_logged_in_admin.rb    

shared_context 'a logged in admin' do 
  before 'create and login admin' do 
    user = create(:user, role: 'admin')
    user.confirm!
    login_as(user, scope: :user)
  end
end
```

Since all files located in ```spec/support``` are automatically required in the default ```spec_helper.rb```, we can call the context from any spec. Now when we need to change how a user is logged in, we just have to change the code in our shared context and without having to change our specs. 

