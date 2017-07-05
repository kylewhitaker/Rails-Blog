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
* Open `config/routes.rb`
``` ruby
Rails.application.routes.draw do
  get 'welcome/index'   # HTTP Get Request URI
  root 'welcome#index'  # map requests at root to Welcome controller's index action
end
```
## 5. Getting up and running
* Add new REST resources to `config/routes.rb`
``` ruby
Rails.application.routes.draw do
  get 'welcome/index'
  resources :articles   # Creates defined routes, all RESTful actions, for "articles"
  root 'welcome#index'
end
```
```
$ bin/rails routes

      Prefix Verb   URI Pattern                  Controller#Action
    articles GET    /articles(.:format)          articles#index
             POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show
             PATCH  /articles/:id(.:format)      articles#update
             PUT    /articles/:id(.:format)      articles#update
             DELETE /articles/:id(.:format)      articles#destroy
        root GET    /                            welcome#index
```
### 5.1 Laying down the groundwork
* Navigate to `http://localhost:3000/articles/new` and receive a routing error.
* Route must have a controller in place to serve the request. Create the `ArticlesController`:
```
$ bin/rails generate controller Articles
```
* Refresh `/articles/new` and receive a different error: unknown action 'new'.
* Add a 'new' action in `app/controllers/articles_controller.rb`:
``` ruby
class ArticlesController < ApplicationController
  def new
  end
end
```
* Refresh `/articles/new` and receive (again) a different error: unknown format or template for 'new'.
* Create a new view file at `app/views/articles/new.html.erb`:
``` html
<h1>New Article</h1>
```
* You should now see no errors and the page loads successfully with the 'New Article' h1 heading.
### 5.2 The first form
* Use Ruby's form builder helper method `form_for`.
* Add the following code to `app/views/articles/new.html.erb`:
``` html
<%= form_for :article do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>
```
* Inspect HTML, find action='articles/new'. We actually want to redirect to 'articles' POST.
``` html
<%= form_for :article, url: articles_path do |f| %>
```
* Fill in the form and submit. Error. We must write code for ArticlesController "create" method.
### 5.3 Creating articles
``` ruby
class ArticlesController < ApplicationController
  def new
  end
 
  def create
  end
end
```
* Error gone, but `204 No Content` returned by default if no response specified in code.
``` ruby
def create
  render plain: params[:article].inspect
end
```
* Create a new article and click submit. Browser window should display:
```
<ActionController::Parameters {"title"=>"Harry Potter", "text"=>"A book about a boy."} permitted: false>
```
### 5.4 Creating the Article model
* Models in Rails use a singular name, and their corresponding database tables use a plural name. Rails provides a generator for creating models, which most Rails developers tend to use when creating new models. To create the new model, run this command in your terminal:
```
$ bin/rails generate model Article title:string text:text
```
* With that command we told Rails that we want an Article model, together with a title attribute of type string, and a text attribute of type text. Those attributes are automatically added to the articles table in the database and mapped to the Article model. Rails created a bunch of files, of which we are most interested in two: `app/models/article.rb` and `db/migrate/20170106133411_create_articles.rb`
### 5.5 Running a migration
