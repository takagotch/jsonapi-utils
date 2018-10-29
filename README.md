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


```

```

```

```
```

