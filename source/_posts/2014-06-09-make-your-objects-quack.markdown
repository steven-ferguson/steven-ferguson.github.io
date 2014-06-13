---
layout: post
title: "Make Your Objects Quack"
date: 2014-06-09 10:33:34 -0700
comments: true
categories: [Ruby, OOP, Ruby on Rails]
published: false
---

One of my big breakthroughs in understanding object oriented design in Ruby came when I learned about [duck typing](http://en.wikipedia.org/wiki/Duck_typing). Wikipedia provides a nice definiition of the concept:
> duck typing is a style of typing in which an object's methods and properties determine the valid semantics, rather than its inheritance from a particular class or implementation of an explicit interface.

This means that in Ruby it does not matter if an object is a specific type or class, only that it can respond to a given message. For example, in the ```stringify``` method below, it does not matter if the argument is an integer or symbol or some other class, only that it can respond to the ```to_s``` method.  

```ruby
def stringify(a)
  a.to_s
end

stringify(1)
=> "1"

stringify(:hello)
=> "hello"

class NewClass
  def to_s
    "works for custom objects too!"
  end
end

stringify(NewClass.new)
=> "works for custom objects too!"
```

Not having to care about a specific object's class enables a lot of flexiblity for swaping out collaborator objects. To illustrate this, let's take a look at an example:

###File Storage with Duck Typing

My app generates reports, and I would like to save these reports. Initially, I just want to save the reports to my local disk.

```ruby

class Report
  def initialize(name, data)
    @name = name
    @data = data
  end

  def save_to(storage)
    storage.store(@name, @data)
  end
end

class FileStorage
  def store(file_name, data)
    File.open("reports/#{file_name}", 'w') do |file|
      file.write(data)
    end
  end
end

report = Report.new('Sales Figures', 'sales data blah...')
file_store = FileStorage.new
report.save_to(file_store)
```

Later, I decide that it would also be nice to have the ability to upload reports to S3. With duck typing, implementing this new feature is as simple as creating a new class for uploading to S3 which implements a ```store``` method:  

```ruby

class S3Storage
  def initialize(bucket_name)
    @bucket = AWS::S3.new.buckets[bucket_name]
  end

  def store(file_name, data)
    @bucket.objects.create(file_name, data)
  end
end

report = Report.new('Sales Figures', 'sales data blah...')
s3_store = S3Storage.new('my_bucket')
report.save_to(s3_store)
```

Start thinking about objects in terms of the messages they receive, and you will be rewarded with the flexibility that duck typing allows.