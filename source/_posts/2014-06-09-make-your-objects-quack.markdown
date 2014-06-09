---
layout: post
title: "Make Your Objects Quack"
date: 2014-06-09 10:33:34 -0700
comments: true
categories: [Ruby, OOP, Ruby on Rails]
published: false
---

One of my big breakthroughs in understanding object oriented design in Ruby came when I learned about duck types. **Get definition from POODR**. They are a powerful technique because they allow a developer to pass a message to a collaborator object without having to worry about the internals of how the collaborator deals with the message. 

In practical terms, this means that I can have objects whose internals are very different play the same collaborator role. To illustrate how this can imporve the resuability of our code let's take a look at an example.

###File Storage Example

Our company would like to receive weekly reports via email attachments from one of our vendors and automatically save these reports for use in our app. To receive inbound emails in our Rails app we will use the [mandrill-rails gem](https://github.com/evendis/mandrill-rails). This really is not important for this example except to know that we will have access to the inbound email via the ```handle_inbound``` method on an email processor controller:

    class EmailProcessorController < ApplicationController
      def handle_inbound(event_payload)
        # we will have access to the attachments here with 
        # event_payload.attachments
      end
    end

For now let's write the attachments to our local disk:

    class EmailProcessorController < ApplicationController
      def handle_inbound(event_payload)
        event_payload.attachments.each do |attachment|
          AttachmentProcessor.new(attachment).process
        end
      end
    end

    class AttachmentProcessor
      def initialize(attachment)
        @attachment = attachment
      end

      def process
        File.open(Rails.root.join('reports', attachment.name), 'w') do |file|
          file.write(attachment.content)
        end
      end
    end

Now a new requirement comes along that CSV reports need to be uploaded to S3 instead of saved on the local disk. Our initial implementation looks like this:

    class AttachmentProcessor
      def initialize(attachment)
        @attachment = attachment
      end

      def process
        if processing_csv?
          upload_to_s3
        else
          save_to_disk
        end
      end

      def upload_to_s3
        AWS::S3.new.buckets['csv_uploads'].objects.create(@attachment.name, @attachment.content)
      end

      def save_to_disk
        File.open(Rails.root.join('reports', @attachment.name), 'w') do |file|
          file.write(@attachment.content)
        end
      end

      def processing_csv?
        @attachment.name.end_with?('.csv')
      end
    end






   
