---
layout: post
title: "Continuous Deployment"
date: 2014-02-17 12:08:47 -0800
comments: true
categories: [Travis CI, Heroku, Agile]
---

I have been hearing a lot recently about [continuous deployment](http://my.safaribooksonline.com/book/web-development/web-services/9781449377465/continuous-deployment/continuous_deployment#X2ludGVybmFsX0h0bWxWaWV3P3htbGlkPTk3ODE0NDkzNzc0NjUlMkZ0aGVfcXVhbGl0eV9kZWZlbmRlcnNfYXBvc3Ryb3BoeV9sYW1lbnQmcXVlcnk9) especially in the context of building a lean startup, so I decided to try and get one of my projects running with continuous deployment.

####Here's what I learned today:

1. [Travis CI](https://travis-ci.org/) is an open source tool for continuous integration and deployment.

2. There are several steps to setting up your project with Travis CI:

   * The [getting started guide](http://docs.travis-ci.com/user/getting-started/) does a good job explaining the basics.

   * For my project, I needed to make a few customizations the biggest one was creating a test database that my test could use. Here is my `.travis.yml`:

        
                  language: ruby
                  rvm:
                  - 2.0.0
                  env:
                  - DB=postresql
                  script:
                  - bundle exec rake db:test:prepare --trace
                  - bundle exec rspec
                  before_script:
                  - psql -c 'create database my_app_test;' -U postgres 
        

3. Travis CI [integrates nicely with Heroku](http://docs.travis-ci.com/user/deployment/heroku/), so that your code be deployed after a successful build. I was having some problems deploying using Anvil, so I switched to using git. Here is what I added to my `.travis.yml` for deploying to Heroku:

        
        deploy:
          provider: heroku
          api_key:
            secure: encrypted-key
          strategy: git
          app: epicodus-qa
          run: "rake db:migrate"
        