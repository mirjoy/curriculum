---
layout: page
title: Task Manager
---

Let's use Sinatra to build an application where we can manage our tasks. 

### Getting Configured

Let's make a project folder from the command line: `mkdir task_manager`.

We'll also need a Gemfile: `touch Gemfile`. Inside of your Gemfile, add Sinatra and Shotgun. [Shotgun](https://github.com/rtomayko/shotgun) will allow us to make changes to our code base without having to restart the server each time. 

```ruby
source 'https://rubygems.org'

gem 'sinatra', require: 'sinatra/base'
gem 'shotgun'

```
We will be using the [modular](http://www.sinatrarb.com/intro.html#Modular%20vs.%20Classic%20Style) style of Sinatra app, which is why we need to require 'sinatra/base'.

Next, let's make a config file from the command line: `touch config.ru`. This file will be used by Rackup. 

```ruby
require 'bundler'
Bundler.require

$LOAD_PATH.unshift(File.expand_path(File.dirname(__FILE__) + "/app"))

require 'controllers/task_manager_app'

run TaskManagerApp
```

The first two lines of this file allow all of your gems to be required. Then, we change the load path so that everything inside of our `app` folder (we haven't created it yet) can be required. Next, we require a file called `task_manager_app` that will be inside of our `controllers` folder. Finally, we call the run method and specify that our app is called TaskManagerApp.

Go ahead and run `bundle install` from the command line. 

### Project Folder Structure

So far, we just have a Gemfile, config.ru, and Gemfile.lock. Let's create the folders we'll need for our Task Manager. 

```
$ mkdir app
$ mkdir db
```

We'll use the `app` folder for all of our implementation code. Our `db` folder will hold our fake database (we're going to use YAML, not a real database).

We'll need a few folders inside of our app folder so that we can separate our files.

```
$ mkdir app/controllers
$ mkdir app/models
$ mkdir app/views
```

Although we could put all of our code inside of the same folder (or even most of it in the same file), we're going to use this structure to mimic the [MVC](http://www.codelearn.org/ruby-on-rails-tutorial/mvc-in-rails) setup that Rails will give us. 

### Getting the App Running

Ok, so we have our project structure. Let's now get our app up and running! Make a file inside of app/controllers called `task_manager_app.rb`:

```
$ touch app/controllers/task_manager_app.rb
```

Inside of it, add the following code:

```ruby
class TaskManagerApp < Sinatra::Base
  set :root, File.join(File.dirname(__FILE__), '..')

  get '/' do
    'hello, world!'
  end
```

Remember when we wrote `run TaskManagerApp` in our `config.ru` file? Well, the class we just defined is what that line in our config.ru refers to.

Line 2 sets the root of our project. Here, we're taking the current file (`app/controllers/task_manager_app`) and going one folder up the chain. This should take us to our `app` folder. The reason we're doing this is because Sinatra will look relative to our app for views and stylesheets. We don't want to put these things in our controller folder, so we're specifying that our root is just `app`.

Next, we tell our app that when a `get` request is sent to the `'/'` path, it should send back 'hello, world!' as the response. Let's try it!

From the command line, run:

```
$ shotgun
```

Navigate to http://localhost:9393/ and you should see magic!

### Using Views

Let's change our controller to render an ERB view instead of 'hello, world!'. Inside of `task_manager_app.rb`:

```ruby
class TaskManagerApp < Sinatra::Base
  set :root, File.join(File.dirname(__FILE__), '..')

  get '/' do
    erb :dashboard
  end
```

This piece of code will look for an .erb file called 'dashboard' in the views folder at the root of the app. Since we've already set the root of the app and created a views folder there, we're ready to make a dashboard file:

```
$ touch app/views/dashboard.erb
```

Inside of this file, let's add links to some functionality we might want in our app:

```erb
<h1>Welcome to the Task Manager</h1>

<ul>
  <li><a href="/tasks">Task Index</a></li>
  <li><a href="/tasks/new">New Task</a></li>
</ul>
```

We have an h1 tag for our welcome message, then an unordered list (ul) with two list items (li) inside. If you are unfamiliar with HTML tags, try one of the [HTML tutorials](https://github.com/turingschool/intermission-assignments/blob/master/prep-for-module-2.markdown) before continuing. 

Inside of each li tag, we have an `a` tag. The href of the tag is the path where the link will go. In the first a tag, the path will be 'http://localhost:9393/tasks'. The second path will be 'http://localhost:9393/tasks/new'. 

Refresh the page. You should see our welcome message and two links. We haven't set up our controller to handle either of these yet, so clicking on these should give us a "Sinatra doesn't know this ditty" error. 

### Adding a Task Index Route

In a Sinatra app, we can add routes by combining an HTTP verb (get, post, put, delete, etc.) with a URL pattern. If you're unfamiliar with HTTP verbs, check out [A Beginner's Guide to HTTP and REST](http://code.tutsplus.com/tutorials/a-beginners-guide-to-http-and-rest--net-16340). 

Our controller currently has one route:

```ruby
  get '/' do
    erb :dashboard
  end
```

Let's add a route for the first link we want -- our Task Index. In `app/controllers/task_manager_app.rb`:

```ruby
class TaskManagerApp < Sinatra::Base
  set :root, File.join(File.dirname(__FILE__), '..')

  get '/' do
    erb :dashboard
  end

  get '/tasks' do
    @tasks = ["task1", "task2", "task3"]
    erb :index
  end
```

What are we doing here? Well, we create an instance variable `@tasks` and assign an array of three strings to it. Then, we render the `index.erb` file. Our instance variable will be available to use in the view. 

Let's try rendering this array in the view. First, we need to create our `index.erb`:

```
$ touch app/views/index.erb
```

Inside of the view, we will iterate through the array and display each string:

```erb
<h1>All Tasks</h1>

<% @tasks.each do |task| %>
  <h3><%= task %></h3>
<% end %>
```

Navigate to 'http://localhost:9393/tasks' and check that each task is displayed. Our `index.erb` is looking ok right now.

### Adding a New Task Route

We need a route that will bring a user to a form where they can enter a new task. This is the second link we had in our dashboard. In our controller:

```ruby
class TaskManagerApp < Sinatra::Base
  set :root, File.join(File.dirname(__FILE__), '..')

  get '/' do
    erb :dashboard
  end

  get '/tasks' do
    @tasks = ["task1", "task2", "task3"]
    erb :index
  end

  get '/tasks/new' do
    erb :new
  end
```

We don't need any instance variables here; we just need to render the view. Let's make that view:

```
$ touch app/views/new.erb
```

In that file, we'll add a form:

```erb
<form action="/tasks" method="post">
  <p>Enter a new task:</p>
  <input type='text' name='task[title]'/><br/>
  <textarea name='task[description]'></textarea><br/>
  <input type='submit'/>
</form>
```

Here we have a form with an action (url path) of `/tasks` and a method of `post`. This combination of path and verb will be important when we create the route in the controller. We then have an text input field for the title, a textarea for the description, and a submit button. 

Navigate to `http://localhost:3000/tasks/new` to see your beautiful form!

Try clicking the submit button. You should get a "Sinatra doesn't know that ditty" error because we haven't set up a route in our controller to handle this action - method combination from the form submission.

In our controller:

```ruby
class TaskManagerApp < Sinatra::Base
  set :root, File.join(File.dirname(__FILE__), '..')

  get '/' do
    erb :dashboard
  end

  get '/tasks' do
    @tasks = ["task1", "task2", "task3"]
    erb :index
  end

  get '/tasks/new' do
    erb :new
  end

  post '/tasks' do
    p "Here are all of the params: #{params}"
    p "Here are the task params: #{params[:task]}"
  end
```

Why `post` instead of `get`? First, we specified a method of post in our form (go look at the form if that sentence doesn't make sense to you). Although we could make it work using `get`, HTTP convention specifies that a `get` request should request data from a specified resource while a `post` should submit data to be processed. In our case, we are submitting form data that needs to be processed, so we're using `post`. 

Inside of this route, we'll need to eventually do some work. But for right now, we're just putting out all of the params and then just the task params.

Go back to `http://localhost:9393/tasks/new`. Fill in a fake title and a fake description. Click submit. You'll see an error, but what we care about right now is our server log. Look at the Terminal window where you ran shotgun. Find the lines where those puts statements ran. You should see something like:

