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
* Rails makes database migrations (e.g. table creation) simple and easy, and they are reversable.
```
$ bin/rails db:migrate
```
* Rails executes migrations in development according to settings in `config/database.yml`. To run migrations in Production, use:
```
$ bin/rails db:migrate RAILS_ENV=production
```
### 5.6 Saving data in the controller
* Update create method in ArticlesController to use model to save new articles to the database.
``` ruby
def create
  @article = Article.new(params[:article])
  @article.save
  redirect_to @article
end
```
* Save a new article. Error: Forbidden Attributes. Rails security feature "strong parameters".
* Refactor create method to be:
``` ruby
@article = Article.new(params.require(:article).permit(:title, :text))
```
* Next, refactor out so parameter permission can be used by multiple methods, such as create & update.
``` ruby
def create
  @article = Article.new(article_params)
  @article.save
  redirect_to @article
end

private
  def article_params
    params.require(:article).permit(:title, :text)
  end
```
### 5.7 Showing articles
* Add the *show* action in `app/controllers/articles_controller.rb`
``` ruby
def show
  @article = Article.find(params[:id])
end
```
* Create a new *view* file `app/views/articles/show.html.erb`
``` html
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
```
* Create a new article: http://localhost:3000/articles/new
* Show an article: http://localhost:3000/articles/1
### 5.8 Listing all articles
* Add the *index* action in `app/controllers/articles_controller.rb`
``` ruby
def index
  @articles = Article.all
end
```
* Create a new *view* file `app/views/articles/index.html.erb`
``` html
<h1>Listing articles</h1>
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
  </tr>
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
    </tr>
  <% end %>
</table>
```
* Now, view all the articles: http://localhost:3000/articles
### 5.9 Adding links
* Add a link in `app/views/welcome/index.html.erb` on welcome page to view all articles
``` html
<h1>Hello, Rails!</h1>
<%= link_to 'My Blog', controller: 'articles' %>
```
* Rails' built-in view helper **link_to** takes display text and path
* Add a link on welcome page to create a new article
``` html
<%= link_to 'New Article', new_article_path %>
```
* Now, add a link (beneath the form) to go back to articles index in `app/views/articles/new.html.erb`
``` html
<%= link_to 'Back', articles_path %>
```
* Finally, add a link to go from single article view back to articles index in `app/views/articles/show.html.erb`
``` html
<%= link_to 'Back', articles_path %>
```
* No need to specify controller if linking to action in same controller (Rails default)
### 5.10 Adding Some Validation
* Rails provides data validation sent to models. Within `app/models/article.rb` add:
``` ruby
validates :title, presence: true,
                  length: { minimum: 5 }
```
* Rails validation covered in detail [here](http://guides.rubyonrails.org/active_record_validations.html).
* With validation in place, **@article.save** will return **false**. Within `app/controllers/articles_controller.rb` add:
``` ruby
def new
  @article = Article.new
end
 
def create
  @article = Article.new(article_params)
  if @article.save
    redirect_to @article
  else
    render 'new'
  end
end
```
* Instead of **redirect_to**, we use **render** which passes the new article arguments into another new article request form.
* Produce an error message if the user makes a bad request for a new article in `app/views/articles/new.html.erb`
``` html
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@article.errors.count, "error") %> prohibited
      this article from being saved:
    </h2>
    <ul>
      <% @article.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```
* The reason why we added **@article = Article.new** in the ArticlesController is that otherwise **@article** would be *nil* in our view, and calling **@article.errors.any?** would throw an error.
* Rails automatically wraps fields that contain an error with a div with class field_with_errors. You can define a css rule to make them standout.
### 5.11 Updating Articles
* Add an *edit* action (between *new* and *create*) to the ArticlesController
``` ruby
def edit
  @article = Article.find(params[:id])
end
```
* Create a new *view* file called `app/views/articles/edit.html.erb`
``` html
<h1>Edit article</h1>
<%= form_for(@article) do |f| %>
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
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
<%= link_to 'Back', articles_path %>
```
* Add an *update* method (between *create* and the private method) in `app/controllers/articles_controller.rb.`
``` ruby
def update
  @article = Article.find(params[:id])
  if @article.update(article_params)
    redirect_to @article
  else
    render 'edit'
  end
end
```
* Finally, show a link to the *edit* action via `app/views/articles/index.html.erb` and make it appear next to the *show* link
``` html
<% @articles.each do |article| %>
  <tr>
    <td><%= article.title %></td>
    <td><%= article.text %></td>
    <td><%= link_to 'Show', article_path(article) %></td>
    <td><%= link_to 'Edit', edit_article_path(article) %></td>
  </tr>
<% end %>
```
* And also add an *edit* link in `app/views/articles/show.html.erb` at the bottom so that there's also an *edit* link on the article's page
``` html
<%= link_to 'Edit', edit_article_path(@article) %> |
<%= link_to 'Back', articles_path %>
```
