---
layout: post
title: "Testing Outgoing Command Messages With RSpec"
date: 2014-03-09 13:10:30 -0700
comments: true
categories: [RSpec, Bogus, TDD, Testing]

keywords: "rspec, mocks, rails, testing"
description: "How to properly unit test your outgoing command messages in ruby"
---

As the size and complexity of the applications I have been working on has grown, the importance of writing good unit test has become increasingly apparent. The goal is to have a test suite that is fast and able to withstand changes.

One of the best approaches I have found to writing good unit tests is discussed by Sandi Metz in [Practical Object Oriented Design in Ruby](http://www.amazon.com/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330) and also in a [talk she gave at RailsConf in 2013](https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf).

She has a rule about testing outgoing command messages that says, _"expect to send outgoing command messages"_. This means that we should assert that the method actually issues the command message, but we don't assert the result of that message.

For example, suppose I have a party, and I want users to be able to RSVP: 

```ruby
class User
  def rsvp(party)
    party.add_guest(self)
  end
end
```
I want to assert that when ```user.rsvp(party)``` is called, the ```add_guest``` message is sent to the party object. Mocks are the perfect tool for testing these command messages because they allow us to test that the correct message is being passed without worrying about how the other object actually deals with that message:

```ruby
describe User do
  describe '#rsvp' do 
    it "tells the party to add the user as a guest" do 
      user = User.new
      party = double('party')
      expect(party).to receive(:add_guest).with(user)
      user.rsvp(party)
    end
  end
end
```
This allows us to test the ```rsvp``` method without having to actually deal with the Party class' implementation of ```add_guest```. But what happens if we change the public interface for ```add_guest```to allow a guest specify how many to friends they plan to bring to the party?

```ruby
class Party
  def add_guest(user, number_of_companions)
    ...
  end
end

class User
  def rsvp(party)
    party.add_guest(self)
  end
end
``` 

Our ```rsvp``` method is now broken because the ```add_guest``` method requires two arguments, but we are only passing it one. Unfortunately our test for ```rsvp``` still passes because our test double does not reflect the actual interface that it is mocking. 

To solve this problem I recently started using a mocking framework called [Bogus](https://github.com/psyho/bogus). It ensures that your test doubles have the same interface  as the real class.

```ruby
require 'bogus/rspec'

...
  describe '#rsvp' do 
    it "tells the party to add the user as a guest" do 
      user = User.new
      party = fake(:party)
      user.rsvp(party)
      expect(party).to have_received.add_guest(user)
    end
  end
...
```

Now when we run the test we receive the following failure:

```
Failures:

  1) User#rsvp tells the party to add the user as a guest
     Failure/Error: party.add_guest(self)
     ArgumentError:
       wrong number of arguments (1 for 2)
``` 

Great! Our test double now implements the same interface as the actual Party class and alerts us that our ```rsvp``` method is not passing the correct number of arguments. Let's update ```rsvp```:

```ruby
class User
  def rsvp(party)
    party.add_guest(self, 0)
  end
end
```
And run our test:

```
Failures:

  1) User#rsvp tells the party to add the user as a guest
     Failure/Error: expect(party).to have_received.add_guest(user)
     ArgumentError:
       tried to stub add_guest(user, number_of_companions) with arguments: #<User:0x007fa49210caa0>
```

Whoops! Looks like our stub for add_guest doesn't implement the correct interface for ```add_guest```. Let's update it:

```ruby
...
  describe '#rsvp' do 
    it "tells the party to add the user as a guest" do 
      ...
      expect(party).to have_received.add_guest(user, 0)
      ...

Finished in 0.006 seconds
1 example, 0 failures
``` 

Another great feature of Bogus is that it supports mocking a duck type. For example, suppose that in addition to parties, we want users to be able to rsvp to dinners. We can implement this with a duck type by creating an ```add_guest``` method for our new Dinner class:

```ruby
class Dinner
  def add_guest(user, number_of_companions)
  ...
  end
end

class User
  def rsvp(guestable)
    guestable.add_guest(self, 0)
  end
end
```

And now in our test we can mock our duck type interface:

```ruby
describe User do
  describe '#rsvp' do 
    it "tells the guestable to add the user as a guest" do 
      user = User.new
      guestable = fake(:guestable) { [Party, Dinner] }
      user.rsvp(guestable)
      expect(guestable).to have_received.add_guest(user, 0)
    end
  end
end
```

Mocks are a great way to focus on the messages that your objects are sending when testing. Give them a try and experience the benefits of a faster more resilient test suite. 