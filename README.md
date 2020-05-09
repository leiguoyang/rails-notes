# Rails notes

## Table of content

- [Controller](#controller)
- [Migration](#migration)
- [Router](#router)
- [Commands](#commands)
- [Sign up](#sign-up)
- [Log in](#log-in)
- [Use Bootstrap in Rails](#use-bootstrap-in-rails)
- [Authentication and authorization](#authentication-and-authorization)
- [Use carrierwave](#use-carrierwave)
- [Resize images](#resize-images)
- [Associations](#associations)

## Controller

When you use the command to create a controller, a view directory also created for your controller, like `app/views/home`.

```bash
$ rails g controller Home
```

And you will see this lines on your screen.

```
Running via Spring preloader in process 2006
      create  app/controllers/home_controller.rb
      invoke  erb
      create    app/views/home
      invoke  test_unit
      create    test/controllers/home_controller_test.rb
      invoke  helper
      create    app/helpers/home_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/home.coffee
      invoke    scss
      create      app/assets/stylesheets/home.scss
```

## Migration
Migration就是对数据库进行更改。有两个作用。

- 对数据库的schema进行修改。如增加新的table或在已有的table上增加新的column。
- 对数据库的data进行修改，如增加新的record.

`bin/rails db:migrate`就是进行migration.

`bin/rails db:rollback`就是撤销上一次的migration.

If you would like to execute migrations in another environment, for instance in production, you must explicitly pass it when invoking the command:

`bin/rails db:migrate RAILS_ENV=production`

## Router
Router有两个目的。

- 指定对应的action来处理request。
- 创建route helper供controller和view来使用, 如`new_user_path`, `new_user_url`

### Route的格式

如有一个叫`Welcome`的controller，里面包含一个叫`first_message`的action，那么产生的path helper是`welcome_first_message_path`。即格式为`controllerName_actionName_path`

Prefix | Verb | URI Pattern | Controller#Action
--- | --- | --- | ---
welcome_index | GET | /welcome/index(.:format) | welcome#index
welcome_first_message | GET | /welcome/first_message(.:format) | welcome#first_message

## Commands
- `bin/rails server` 运行rails程序
- `bin/rails server -p $PORT -b $IP` 在一个特定的port或IP运行
- `rails routes` 列出所有产生的routes

## Sign up
Sign up其实就是创建新的用户，所以需要一个`User` model.

`rails g model user email:string password_digest:string`

`rails db:migrate`

Add bcrypt (~> 3.1.7) to Gemfile to use `has_secure_password`. `has_secure_password`的具体用法请参考Rails API说明。

```ruby
gem 'bcrypt', '~> 3.1.7'
```

And then run `bundle` to install the gem.

在`user.rb`里使用`has_secure_password`。

```ruby
class User < ApplicationRecord
  has_secure_password

  validates :email, presence: true, uniqueness: true
end
```

UsersController.

`rails g controller users`

`controllers/users_controller.rb`

```ruby
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
  end
  
  def new
    @user = User.new
  end
  
  def create
    @user = User.new(user_params)
    if @user.save
      # if successfully create the user account, show the user's homepage
      @is_successfully_created = true
      
      redirect_to @user
    else
      render "new"
    end
  end
  
  private
    def user_params
      params.require(:user).permit(:email, :password)
    end
end
```

`views/users/new.html.erb`

```html
<h1>Sign Up</h1>

<%= form_for @user do |f| %>
  <% if @user.errors.any? %>
    <div class="error_messages">
      <h2>Form is invalid</h2>
      <ul>
        <% @user.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.text_field :email %>
  </div>
  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password %>
  </div>
  <div class="actions"><%= f.submit "Sign Up" %></div>
<% end %>
```

`views/users/show.html.erb`

```html
<h1>My homepage</h1>

<%= puts @is_successfully_created %>

<% if @is_successfully_created %>
  <div>
    Your account is successfully created.  
  </div>
<% else %>
  <div>
    The instance variable <code>@is_successfully_created</code> doesn't exist at all.
  </div>
<% end %>

<p>This is your homepage. Enjoy.</p>
```
## Log in
其实我还不知道什么叫登录，目前理解的是创建一个session ID来存放user的id。而session ID和cookie又怎么关联起来我还没弄懂。

实际上登录是假的。你在Edge登录完后，照样可以在Firefox上登录，说明登录只是将你当前的游览器和服务器进行了一种关联而已。如果在浏览器登录完后在打开一个新的tab尝试去登录发现不行，说明cookie的scope是整个浏览器，而不仅仅是当前的tab.

### Step1 Create a Sessions Controller

```
rails g controller sessions
```

My `sessions_controller.rb` looks like this.

```ruby
class SessionsController < ApplicationController
  
  def new
    if logged_in?
      @message = 'You already logged in.'
      redirect_to root_path
    else
      render 'new'
    end
  end
  
  def create
    unless logged_in?
      user = User.find_by(name: params[:login][:name])
    
      if user
        if user.authenticate(params[:login][:password])
          # Authentication passed
          # The user login successfully
          session[:user_id] = user.id
          @message = '登录成功'
          redirect_to root_path
        else
          @message = '密码不正确'
          render 'new'
        end
      else
        @message = '用户名不正确'
        render 'new'
      end
    end
  end

  def logout
    session[:user_id] = nil
    @message = 'You have successfully logged out.'
    redirect_to root_path
  end
end
```

The log in form `app/views/sessions/new.html.erb` looks like this.

```html
<% if @message %>
<div class="alert alert-info"><%= @message %></div>
<% end %>
<h1 class="mb-5">登录</h1>
<%= form_for :login do |f| %>
  <div class="form-group">
    <%= f.label :name, '用户名' %>
    <%= f.text_field :name, autofocus: true, class: 'form-control' %>
  </div>
  <div class="form-group">
    <%= f.label :password, '密码' %>
    <%= f.password_field :password, class: 'form-control' %>
  </div>
  <div>
    <%= f.submit '登录', class: 'btn btn-primary' %>
  </div>
<% end %>
```

我认为在获取任何一种资源前都必须确认用户是否已登录，所以我定义一个叫	`logged_in`的方法放在`ApplicationController`里，这样每一个controller可以使用。

```ruby
class ApplicationController < ActionController::Base

  def logged_in?
    session[:user_id] != nil
  end

end
```

也许有更合理的放置地方。




## Use Bootstrap in Rails

The following procedure references <a href="https://github.com/twbs/bootstrap-rubygem/blob/master/README.md">https://github.com/twbs/bootstrap-rubygem/blob/master/README.md</a>.

First, add two gems to your 'Gemfile'.

```ruby
# Use bootstrap
gem 'bootstrap', '~> 4.4.1'
gem 'jquery-rails'
```

Run `bundle` to install the gems and restart your server if it is running.

Second, rename `app/asset/stylesheets/application.css` to `app/asset/stylesheets/application.scss`. Remove all the `*= require` and `*= require_tree` statements in this file. Finally, add `@import "bootstrap";` to the file. The file may look like this now.

```scss
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, or any plugin's
 * vendor/assets/stylesheets directory can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any other CSS/SCSS
 * files in this directory. Styles in this file should be added after the last require_* statement.
 * It is generally better to create a new file per style scope.
 *


 */
@import "bootstrap";
```

Third, Add Bootstrap dependencies and Bootstrap to your `application.js`.

```js
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

The file may look like this now.

```js
// This is a manifest file that'll be compiled into application.js, which will include all the files
// listed below.
//
// Any JavaScript/Coffee file within this directory, lib/assets/javascripts, or any plugin's
// vendor/assets/javascripts directory can be referenced here using a relative path.
//
// It's not advisable to add code directly here, but if you do, it'll appear at the bottom of the
// compiled file. JavaScript code in this file should be added after the last require_* statement.
//
// Read Sprockets README (https://github.com/rails/sprockets#sprockets-directives) for details
// about supported directives.
//
//= require rails-ujs
//= require activestorage
//= require turbolinks
//= require_tree .
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

Now you are free to use bootstarp on your project.

## Authentication and authorization

Here is my understanding.

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all.order('created_at DESC')
  end
  
  def show
    @article = Article.find(params[:id])  
  end
  
  def new
    @article = Article.new
  end
  
  def edit
    # before the edit page is shown to the user, must check whether the user who request is the owner of this resource, like an article or a like
    @article = Article.find(params[:id])
    # if user_who_request == resource_owner
    #   send the edit page to the user_who_request
    # else
    #   reject
    # end
    
  end
  
  def create
    @article = Article.new(article_params)
    if @article.save
      render 'show'
    else
      render 'new'
    end
  end
  
  def update
    @article = Article.find(params[:id])
    # before updating the resource, must check whether the user who request is the owner of this resource, like an article or a like
    # I guessing i am doing the authorization now, seems I know more what atuhorizatio is.
    if @article.update(article_params)
      render 'show'
    else
      render 'edit'
    end
  end
  
  private
  def article_params
    params.require(:article).permit(:title, :text)
  end
end
```

## Use carrierwave

- Reference [https://github.com/carrierwaveuploader/carrierwave](https://github.com/carrierwaveuploader/carrierwave)
- 
### Step 1 Add `carrierwave` gem to the `Gemfile`.

```ruby
gem 'carrierwave', '~> 2.0'
```

Then run `bundle` to install it. Restart your server if already running.

### Step2 Add a column for your model in the database to hold the url to the file 

Say you have a `Profile` model and want to upload a picture.

```bash
rails g migration add_picture_to_profiles picture:string
```

Then run `rails db:migrate` to update the database.

### Step3 Create an uploader

```bash
rails g uploader Picture
```

Now a file called `app/uploaders/picture_uploader.rb` shoulb be created.

```ruby
class PictureUploader < CarrierWave::Uploader::Base
  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  # include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # Provide a default URL as a default if there hasn't been a file uploaded:
  # def default_url(*args)
  #   # For Rails 3.1+ asset pipeline compatibility:
  #   # ActionController::Base.helpers.asset_path("fallback/" + [version_name, "default.png"].compact.join('_'))
  #
  #   "/images/fallback/" + [version_name, "default.png"].compact.join('_')
  # end

  # Process files as they are uploaded:
  # process scale: [200, 300]
  #
  # def scale(width, height)
  #   # do something
  # end

  # Create different versions of your uploaded files:
  # version :thumb do
  #   process resize_to_fit: [50, 50]
  # end

  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  # def extension_whitelist
  #   %w(jpg jpeg gif png)
  # end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end
end
```

Step4 Mount your uploader to the model

In `profile.rb` add this line of code.

```ruby
class Profile < ApplicationRecord
  belongs_to :user
  mount_uploader :picture, PictureUploader
end
```

Now you can upload the file and stored it.

?How is the file url is created?

!必须对文件类型进行限制，如上传图片时只允许特定的格式，不然像Android可以将任意的文件进行上传。

## Resize images
!you need to process the picture before uploaded them to the server, for example to resize it.

### Step 1 Install imagemagick
Run this command to install imagemagick to your OS.

```bash
sudo apt-get install imagemagick
```

Then add this gem to your `Gemfile`.

```
gem 'mini_magick'
```

Then run `bundle` to install it.

### Step 2 Edit `app/uploaders/picture_uploader.rb`

```ruby
class PictureUploader < CarrierWave::Uploader::Base
  # Include RMagick or MiniMagick support:
  # include CarrierWave::RMagick
  include CarrierWave::MiniMagick

  # Choose what kind of storage to use for this uploader:
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  # Provide a default URL as a default if there hasn't been a file uploaded:
  # def default_url(*args)
  #   # For Rails 3.1+ asset pipeline compatibility:
  #   # ActionController::Base.helpers.asset_path("fallback/" + [version_name, "default.png"].compact.join('_'))
  #
  #   "/images/fallback/" + [version_name, "default.png"].compact.join('_')
  # end

  # Process files as they are uploaded:
  # process scale: [200, 300]
  
  process resize_to_fit: [500, 500]
  
  # def scale(width, height)
  #   # do something
  # end

  # Create different versions of your uploaded files:
  version :thumb do
    process resize_to_fit: [96, 96]
  end

  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
  # def extension_whitelist
  #   %w(jpg jpeg gif png)
  # end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end
end
```

## Associations

对各个model建立associations的目的是方便存取数据，如`User.followers`,关键还是要先建好表和确定表之间的关联。建表是最基础和最重要的，实际上也没associations这么难理解。要多练多看才能准确使用associations的`has_many`, `belongs_to`这些方法。
