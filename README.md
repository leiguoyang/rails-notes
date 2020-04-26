# Rails notes

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
