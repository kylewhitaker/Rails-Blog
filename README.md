# My Rails Blog

## 1. Guide Assumptions
* Ruby (v.2.2.2 or newer)
* RubyGems (package manager)
* SQLite3 (database)
## 2. What is Rails?
* Rails makes assumptions about best practices for development. It is opinionated software.
* Two guiding principles:
  1. Don't Repeat Yourself (DRY)
  2. Convention over Configuration
## 3. Creating a new Rails Project
### 3.1 Installing Rails
```
$ ruby -v   // verify Ruby installed & version
```
```
$ sqlite3 --version   // verify SQLite3 installed & version
```
```
$ gem install rails   // install Rails
$ rails --version     // verify Rails version
```
### 3.2 Creating the Blog Application
```
$ rails new blog  // "generator" creates directory + application, installs gem dependencies
$ rails new -h    // all generator options
```
## 4. Hello, Rails!
### 4.1 Starting up the Web Server
```
$ cd blog
$ bin/rails server   // starts Puma web server
```
### 4.2 Say "Hello", Rails
* Create a new controller "Welcome" with an action "index":
```
$ bin/rails generate controller Welcome index
```
* Controller: `app/controllers/welcome_controller.rb`
* View: `app/views/welcome/index.html.erb`
* Open the view. Replace content with:
``` html
<h1>Hello, Rails!</h1>
```
### 4.3 Setting the Application Home Page

