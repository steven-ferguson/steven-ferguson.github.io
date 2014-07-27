---
layout: post
title: "Switches with Ruby on Rails"
date: 2014-07-19 13:47:42 -0700
comments: true
published: false
categories: [Ruby on Rails, UX, AJAX] 
keywords: "switches, rails, ajax, bootstrap"
description: "Implementing switches in Ruby on Rails with AJAX"
---

In just about every Rails application that I have built there has been a need to switch a model between one or more states. This post will demonstrate the solution I use to implement this common UX pattern.

###The Setup
I have a Ruby on Rails application that I use to manange my robot minions, and I would like easily turn them on and off from my dashboard view.

{% img https://s3-us-west-2.amazonaws.com/stevenfergusonblog/Screen+Shot+2014-07-26+at+6.56.37+PM.png %}

You can see a live version of [my minion dashboard in action here](http://rails-minions.herokuapp.com/).

I use a partial called ```_minion.html.erb``` to display each of the table rows. I should note if you can't already tell, I'm using bootstrap.

```
#_minion.html.erb

<tr id='<%= dom_id(minion) %>' >
  <td><%= minion.name %></td>
  <td>
    <div class='btn-group'>
      <%= link_to 'Off', minion_turn_off_path(minion), method: :post, class: "btn #{minion.off_button_class}", remote: true %>
      <%= link_to 'On', minion_turn_on_path(minion), method: :post, class: "btn #{minion.on_button_class}", remote: true %>
    </div>
  </td>
</tr>
```
A couple things to note about the markup:

1. I'm using the Rails helper ```dom_id``` to give each row a unique id. This will come in handy later when I need to select the minion with JQuery.

2. Each ```link_to``` has ```remote: true``` set. This tells JQuery-UJS that I want the request to be made as an AJAX request. The ```method: :post``` specifies that I want this request to be made as a POST (the default for link_to is GET). 

3. I use a decorator for my minion models to seprate view specific code from business logic. Here is my minion decorator. You can learn more about the decorator pattern [here](http://robots.thoughtbot.com/tidy-views-and-beyond-with-decorators).

```
class MinionDecorator < Draper::Decorator
  delegate_all

  def off_button_class
    if model.on?
      not_selected_button_class
    else
      selected_button_class
    end
  end

  def on_button_class
    if model.on?
      selected_button_class
    else
      not_selected_button_class
    end
  end

  def selected_button_class
    'btn-primary disabled'
  end

  def not_selected_button_class
    'btn-default'
  end
end
```
So now that the view logic is taken care of let's look at how the router and controller implemen this action.

```
#config/routes.rb

resources :minions, only: [:index] do
  post 'turn_on'
  post 'turn_off'
end
```

Here we define two custom routes to handle the switching the minions on and off. These translate to urls of ```/minions/:minion_id/turn_on``` and ```/minions/:minion_id/turn_off``` respectively. I think these URLs most accurately represent the actions. These routes correspond to the ```turn_on``` and ```turn_off``` methods in the controller.

```
#controllers/minions_controller.rb

...
def turn_on
  power_switch_change(:turn_on)
end

def turn_off
  power_switch_change(:turn_off)
end

private
def power_switch_change(action)
  @minion = Minion.find(params[:minion_id])
  @minion.send(action)
  @minion = @minion.decorate

  respond_to do |format|
    format.html { redirect_to minions_path }
    format.js { render 'power_switch_change' }
  end
end
```

And finally the Server-generated JavaScript Response (SJR):

```
#views/minions/power_switch_change.js.erb

$("#<%= dom_id(@minion) %>").replaceWith("<%= j render @minion %>");
```

Further Reading:
* [DHH on SJR](https://signalvnoise.com/posts/3697-server-generated-javascript-responses)
* [Implementing a favorite button with AJAX](http://www.topdan.com/ruby-on-rails/ajax-toggle-buttons.html)