---
layout: post
title: "My 'Aha' moments from the weekend"
date: 2014-02-23 11:35:24 -0500
comments: true
categories: 
---
This past week was a bit hectic for me considering the pace picked up at the Flatiron School and on top I had to prepare for a [meetup](http://www.meetup.com/nyc-on-rails/events/166756172/). I managed to make it through the week and after getting 12 hrs of much needed sleep on Saturday I went about reviewing the recorded lectures. By revisiting these lectures I had several 'Aha' moments and would like to share them. I apologize in advance if you already know this but for me...



![image](../images/jackie-chan-mind-blown.jpeg)


###1. Modules should have abstract values
Don't rely on literal values! A major problem with relying on literal values is scope. In the example below on line 4 the **@@all** variable should be encapsulated in a method.

```ruby
module Rankable
  module ClassMethods
    def top_5
      @@all.sort_by {|s|s.rank}[0..4]
    end
  end
end
```
The fix is to encapsulate **@@all** in a method resulting in the following code.

```ruby
module Rankable
  module ClassMethods
    def top_5
      self.all.sort_by {|s|s.rank}[0..4]
    end
  end
end
```

###2. Order Matters
When listing the files to load up it should be 

    1. Gems 
    2. Concerns(Modules) 
    3. Models(Classes)

The following code would break because the module line 3 is loaded after the model(Student Class)

```ruby
require 'sqlite3'
require_relative '../lib/student' #class Student
require_relative '../lib/concerns/persistable' #module Persistable
```
The fix is to load it in the correct order.
```ruby
require 'sqlite3'
require_relative '../lib/concerns/persistable' #module Persistable
require_relative '../lib/student' #class Student
```

###3. Be Weary of Class Variables**(@@)** 
Class variables can leak due to inheritance. For example in the following code when a new Dog instance is created the @@breeds will be overwritten to **[:affenpoo]**. 

```ruby
class Dog
  @@breeds = [:poodle, :pug]
end

class Poodle < Dog
  @@breeds = [:affenpoo]
end
```

To avoid possible leaks, class CONSTANTS should be used. The following would fix the problem above and when a new Dog instance is created the BREEDS constant in the Dog class will remain **[:poodle, :pug]**. Likewise the BREEDS constant in the Poodle class will remain **[:affenpoo]**.

```ruby
class Dog
  BREEDS = [:poodle, :pug]
end

class Poodle < Dog
  BREEDS = [:affenpoo]
end
```
###4. All Methods Must Have a Receiver
All methods must have a receiver and if not defined it is ***self***.

```ruby
class Movie
  attr_accessor :name
  def save
    DB.execute("INSERT INTO movies(name) VALUES (?)", name)
  end
end
``` 
In line 4 the last argument **'name'** is not a variable. The way ruby determines this is by first looking for a variable called 'name'. If not defined it then looks for method called name which is defined on line 2 **"attr_accessor :name"**. 

###5. Found Another Tool - The Tap Method
What the tap method does is yields x to the block and then returns x.
```ruby
def self.new_from_db(row)
  s=self.new
  s.id=row[0]
  s.name=row[1]
  s.tagline=row[2]
  s
end
```
Using the tap method
```ruby
def self.new_from_db(row)
  self.new.tap do |s|
    s.id=row[0]
    s.name=row[1]
    s.tagline=row[2]
  end
end
```




