---
layout: post
title: "Revisiting Mass Assignments with Params"
date: 2014-03-08 23:25:26 -0500
comments: true
categories: 
---
Considering there isn't much information on params I wanted to write a blog post on it. However during the past week at the Flatiron School, we were forwarded two informative blog posts on params. You can check them out here:

http://codefol.io/posts/How-Does-Rack-Parse-Query-Params-With-parse-nested-query

http://surrealdetective.github.io/blog/2013/07/01/the-nested-ruby-params-hash-for-complex-html-forms-and-sinatra/


Rather than reinvent the wheel I decided to add to the material mentioned in the blog posts above. One of the key points we learned during lecture was to use proper naming conventions so that we could do a mass assignment with params. For example:

```ruby users_controller.rb
class UsersController < ApplicationController

  get '/users/new' do
    erb :"users/new.html"
  end

  post '/users' do

      @user = User.new
      @user.name = params[:name]
      @user.email = params[:email]
      @user.password = params[:password]
      @user.save

      redirect "/users/#{@user.id}"
  end

  get '/users/:id' do
    @user = User.find(params[:id])
    erb :"users/show.html"
  end

end
```
Using mass assignment, lines 10-12 above can be replaced with `@user=User.new(params[:user])` resulting in the following.

```ruby users_controller.rb
class UsersController < ApplicationController

  get '/users/new' do
    erb :"users/new.html"
  end

  post '/users' do

      @user = User.new(params[:user])
      @user.save

      redirect "/users/#{@user.id}"
  end

  get '/users/:id' do
    @user = User.find(params[:id])
    erb :"users/show.html"
  end

end
```
There are only three attributes(name, email, password) but as attributes are added one can quickly see how mass assignment will be useful. 

Using the same example, the trick to utilizing mass assignment is to get params to return a hash with the following structure:

```
params == {
            :user => {
              :name => "John Doe", 
              :email => "john.doe@flatironschool.com",
              :password => "123456"
            }
          }
``` 

 To get params to have that type of structure, one needs to properly name the form inputs for the name attribute.

```html new.html.erb
<form action="/users" method="POST">
  <p>
    <label for="name">Name</label><br/>
    <input type="text" name="user[name]" id="name" />
  </p>
  <p>
    <label for="email">Email</label><br/>
    <input type="text" name="user[email]" id="email" />
  </p>
  <p>
    <label for="pwd">Password</label><br/>
    <input type="text" name="user[password]" id="pwd" />
  </p>
   <input type="submit" value="Submit" />
</form>
```
Notice within the **\<input>** controls: `name="user[name]"`, `name="user[email]"`, `name="user[password]"`. Using this naming convention we are able to get back the desired params hash because both **user** and **[attribute]** are keys to the input value.

Also because our naming covention will return a hash - the order of which the form labels are arranged does not affect the mass assignment. However, the last key value must be identical to the column name of the attribute assigned.

Based on the naming convention above our migration table should look like below

(_Note_: I mixed up the order of the columns to show that order doesn't matter but the name attributes **must** match the last key of the params hash)

```ruby 01_create_users.rb
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :password  #order does not have to match the <form> inputs 
      t.string :email
      t.string :name 
      t.timestamp
    end
  end
end
```

Column names must match the last key of the params hash for mass assignment. If I named the attribute **pwd** `<input type="text" name="user[pwd]">` while the table column is named `:password` I would get the following error:

![image](http://jisukim82.github.io/images/unknown_attribute.png)


Thus the column names match up with the last key of the params hash.

1. Line 6 `:name` matches the last key `<input type="text" name="user[name]"/>`

2. Line 5 `:email` matches the last key `<input type="text" name="user[email]"/>`

3. Line 4 `:password` matches the last key `<input type="text" name="user[password]"/>`














