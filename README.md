# Easy Roles

Simple rails gem for basic role authorization with ruby on rails.

## Changelog

Please read the CHANGELOG.md file.

## Install
> :warning: I have been maintaining this gem for over a year now and have not had any of my pull requests approved and the original owner has not gotten back to me about transfering ownership. What this means is that I'll eventually need to host this gem at rubygems.org under a different gem name. To use this gem with your Rails application, instead of installing it from the rubygems.org repository, add the folling to your `Gemfile`:

```ruby
gem 'easy_roles', git: 'https://github.com/aarona/easy_roles.git'
```

## Basic Setup

### Serialize Method

Add the following to your Gemfile:

```ruby
gem 'easy_roles', git: 'https://github.com/aarona/easy_roles.git'
```

Then generate the migration:

```
rails g easy_roles user roles
```

Or add a `roles` column to your users model, and set the default value to `--- []`. Please note you can call this column anything you like, I like to use the name "roles".

```
t.string :roles, default: "--- []"
```

Then you need to add `easy_roles :column_name` to your model:

```ruby
  class User < ActiveRecord::Base
    easy_roles :roles
  end
```

### Bitmask Method

Add the following to your Gemfile:

```ruby
gem 'easy_roles', git: 'https://github.com/aarona/easy_roles.git'
```

Then generate the migration:

```  
rails g easy_roles user roles --use-bitmask-method
```

Or add a `roles_mask` column to your users model of type `integer`, and set the default value to `0`. Please note you can call this column anything you like, I like to use the name "`roles_mask`":

```
t.integer :roles_mask, default: 0
```

Add `easy_roles :column_name, method: :bitmask` to your model:

```ruby
class User < ActiveRecord::Base
  easy_roles :roles_mask, method: :bitmask
end
```

And lastly you need to add a constant variable which stores an array of the different roles for your system. The name of the constant must be the name of your column in full caps.

#### WARNING: Bitmask storage relies that you DO NOT change the order of your array of roles, if you need to add a new role, just append it to the end of the array.

```ruby
class User < ActiveRecord::Base
  easy_roles :roles_mask, method: :bitmask
  
  # Constant variable storing roles in the system
  ROLES_MASK = %w[admin moderator user].freeze
end
```

## Usage

Easy roles extends your model, and adds a few methods needed for basic role authorization.

adding a role to a user

```add_role 'role'```

adding multiple roles at the same time to a user

```add_roles 'admin', 'manager'```

removing a role from a user

```remove_role 'role'```

check to see if a user has a certain role

```ruby
has_role? 'role'
# or
is_role? # role being anything you like, for example 'is_admin?' or 'is_awesome?'
```

For every method above there is a bang method too.

```ruby
  add_role! 'role'
  add_roles! 'admin', 'manager'
  remove_role! 'role'
```

## Examples

```ruby
@user = User.first

@user.add_role 'admin'

@user.is_admin?
=> true

@user.has_role? 'admin'
=> true

@user.is_awesome?
=> false

@user.add_role 'awesome'

@user.is_awesome?
=> true

@user.remove_role 'admin'

@user.is_admin?
=> false

etc etc
```

## Protecting controllers

There are many ways to implement views for specific roles, so I did not specifically supply one. Here's an example on what you could do:

```ruby
class ApplicationController < ActionController::Base
  def admin_required
    unless current_user && current_user.is_admin?
      flash[:error] = "Sorry, you don't have access to that."
        redirect_to root_url and return false
    end
  end
end
```

Then in your `AdminsController` or any controller that you only want admins to view:

```ruby
class AdminsController < ApplicationController
   before_filter :admin_required
end

class MarksController < ApplicationController
   before_filter :admin_required, only: %w(create update)
end
```

## Scopes

By default, easy_roles adds the `with_role` scope to your models.

```ruby
  @admins = User.with_role('admin')
```

If you're using the bitmask method, an `ArgumentError` will be thrown if an undeclared scope is queried. Since an `ActiveRecord::Relation` is returned, the query is chainable:

```log
  BitmaskUser.with_role('admin').where(active: true).to_sql
  # => SELECT "bitmask_users".* FROM "bitmask_users" WHERE "bitmask_users"."roles_mask" IN (1, 3, 5, 7) AND "bitmask_users"."active" = 't'
  
  SerializeUser.with_role('admin').where(active: true).to_sql
  # => SELECT "serialize_users".* FROM "serialize_users" WHERE "serialize_users"."active" = 't' AND (serialize_users.roles LIKE "%!admin!%")
```

easy_roles also supports a `without_role` scope.

```ruby
@non_admins = User.without_role('admin')
```

Follow me on twitter: http://twitter.com/_aaron0

## License

Copyright (c) 2020 Platform45

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
