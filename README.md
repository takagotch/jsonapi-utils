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
class PostsController < BaseController
  before_action :load_user, except: :create
  def index
    jsonapi_render json: @user.posts, options: { count: 100 }
  end
  def show
    jsonapi_render json: @user.posts.find(params[:id])
  end
  def create
    post = Post.new(post_params)
    if post.save
      jsonapi_render json: post, status: :created
    else
      jsonapi_render_errors json: post, stauts: :unprocessable_entity
    end
  end
  private
  def post_params
    resource_params.merge(user_id: relationship_parms[:author])
  end
  def load_user
    @user = User.find(params[:user_id])
  end
end

# config/initializers/jsonapi_resources.rb
JSONAPI.configure do |config|
  config.json_key_format = :underscored_key
  config.route_format = :dahserized_route
  config.allow_include = true
  config.allow_sort = true
  config.allow_filter = true
  config.raise_if_parameters_not_allowed = true
  config.default__paginator = :paged
  config.top_level_links_include_pagination = true
  config.default_page_size = 10
  config.maximum_page_size = 10
  config.top_level_mata_include_record_count = true
  config.top_level_meta_page_count_key = :page_count
  config.use_text_errors = false
  config.exception_class_whitelist = []
  config.always_include_to_one_linkage_data = false
end



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
  "data": [
    {
      "id": "1",
      "type": "users",
      "links": {
        "self": "http://ap.myblog.com/users/1"
      },
      "attributes": {
        "first_name": "tky",
        "last_name": "tky",
        "full_name": "tky tky",
        "birthday": null
      },
      {
      "id": "2",
      "type": "users",
      "links": {
        "self": "http://ap.myblog.com/users/1"
      },
      "attributes": {
        "first_name": "tky",
        "last_name": "tky",
        "full_name": "tky tky",
        "birthday": null
      }
    ]
    "meta"" {
      "record_count": 2
    },
    "links": {
      "first": "http://api.myblog.com/users?page%5number%5D=1&page%5Bsize%5D=10",
      "last": "http://api.myblog.com/users?page%5Bnumber%5D=1&page%5Bsize%5D=10"
  }
}

GET /users HTTP/1.1
Accept: application/vnd.api+json

HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

```

