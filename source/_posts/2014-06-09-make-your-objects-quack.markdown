---
layout: post
title: "Make Your Objects Quack"
date: 2014-06-09 10:33:34 -0700
comments: true
categories: [Ruby, OOP, Ruby on Rails]
published: false
---

One of my big breakthroughs in understanding object oriented design in Ruby came when I learned about duck typing. The concept can be summarized as: "if and object looks like a duck, walks like a duck, and talks like a duck, then it's a duck". It is a powerful technique because it allows you to swap out different types of collaborator objects as long as the honor a specific interface. In practical terms, this means that I can have objects that internally handle a message very differently, but to the outside world, they play the same collaborator role. 

To illustrate how this can imporve the resuability of our code let's take a look at an example.

###File Storage Example

My app generates reports, and I would like to save these reports. Initially, I just want to save the reports to my local disk.

    class Report
      attr_reader :name, :data
      
      ...

      def save_to(storage)
        storage.save(name, data)
      end
    end

    class FileStorage
      def save(file_name, data)
        File.open(Rails.root.join('reports', file_name), 'w') do |file|
          file.write(data)
        end
      end
    end

    report = Report.new('Sales Figures', 'sales data blah...')
    storage = FileStorage.new
    report.save_to(storage)

A new requirement comes in that we would also like the ability to upload reports to S3. With duck typing, this is as simple as creating a new class for uploading to S3 and implementing the storage interface.  

    class S3Storage
      def save(file_name, data)
        #logic for uploading to S3
      end
    end

    report = Report.new('Sales Figures', 'sales data blah...')
    storage = S3Storage.new
    report.save_to(storage)

This design allows us to quickly swap out storage implementations as needed. 
