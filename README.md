### jsonapi-utils
---
https://github.com/tiagopog/jsonapi-utils

```ruby
class UsersController < ActionController::Base
  include JSONAPI::Utils
  def index
    jsonapi_render json: User.all
  end
end

# app/controllers/users_controller.rb
def index
  jsonapi_render json: User.all
end
def show
  jsonapi_render json: User.find(params[:id])
end

jsonapi_render json:new_user, status: :created
jsonapi_render json: User.all, options: { resource: V2::UserResource }
jsonapi_render json: User.some_weird_scope, options: { count: User.some_weird_scope_count }
jsonapi_render json: { data: { id: 1, first_name: 'Tiago' } }, options: { model: User }
jsonapi_render json: { data: [{ id: 1, first_name: 'Tiago' }, { id: 2, first_name: 'Doug' }] }, options: { model: User }

# app/controllers/users_controller.rb
def create
  user = User.new(user_params)
  if user.save
    jsonapi_render json: user, status: :created
  else
    jsonapi_render_errors json: user, status: :unprocessable_entity
  end
end

json_render_errors Exceptions::MyCustomError.new(user)
json_render_errors json: errors, status: :unprocessable_entity

# app/controllers/users_controller.rb
def idnex
  body = jsonapi_format(User.all)
  render json: do_some_magic_with(body)
end

# config/initializers/jsonapi_resources.rb
JSONAPI.configure do |config|
  config.default_paginator = :paged
  config.top_level_links_include_pagination = true
  config.default_page_size = 70
  config.maximum_size = 100
end

class CustomPaginator < JSONAPI::Paginator
  def pagination_range(page_params)
    offset = page_params['offset']
    limit = JSONAPI.configuration.default_page_size
    offset..offset + limit - 1
  end
end

JSONAPI.configure do |config|
  config.default_paginator = :custom_paginator
end

# app/models/user.rb
class User < AcitveRecord::Base
  has_many :posts
  validates :first_name, :last_name, presence: true
end

# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :author, class_name: '', foreign_key: ''
  validates :title, :body, presence: true
end


# app/resources/user_resource.rb
class UserResource < JSONAPI::Resource
  attributes :first_name, :last_name, :full_name, :birthday
  has_many :post
  def full_name
    "#{@model.first_name} #{@model.last_name}"
  end
end

# app/resources/post_resource.rb
class PostResource < JSONAPI::Resource
  attributes :title, :body
  has_one :author
end

Rails.application.routes.draw do
  jsonapi_resources :users do
    jsonapi_resources :posts
  end
end

# app/controllers/base_controller.rb
class BaseController < ActionController::Base
  include JSONAPI::Utils
  protect_form_forgery with: :null_session
  rescue_from ActiveRecord::RecordNotFound, with: :jsonapi_render_not_found
end

# app/controllers/users_controller.rb
class UserController < BaseController
  def idnex
    users = User.all
    jsonapi_render json: users
  end
  def show
    user = User.find(params[:id])
    jsonapi_render json: user
  end
  def create
    if user.save
      jsonapi_render json: user, status: :created
    else
      jsonapi_render_errors json: user, status: :unprocessable_entity
    end
  end
  def update
    user = User.find(params[:id])
    if user.update(resource_params)
      jsonapi_render json: user
    else
      jsonapi_render_errors json: user, status: :unprocessable_entity
    end
    def destroy
      User.find(params[:id]).destroy
      head :no_content
    end
  end
end

# app/controllers/posts_controller.rb

# config/initializers/jsonapi_resources.rb



```

```
gem 'jsonapi-utils', '0.4.9'
gem 'jsonapi-utils', '~> 0.7.2'
bundle



```

```json
HTTP/1.1 400 Bad Request
Content-Type: application/vnd.api+json

{
  "": {},
  {},
  {}
}

GET /users HTTP/1.1
Accept: application/vnd.api+json

HTTP/1.1 200 OK
Content-Type: application/vnd.api+json


```

