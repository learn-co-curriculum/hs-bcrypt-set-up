

## Setting Up Your App with Bcrypt

We want our users to sign in with a password, but we can't just save user passwords in plain text. That makes it too easy for someone to hack in and steal everyone's secrets.

We’re going to set up secure password storage with a gem called [Bcrypt](https://github.com/codahale/bcrypt-ruby) to encrypt our users passwords before we save them in our database.
  
  + Bcrypt uses a hashing algorithm, which means it takes a chunk of data (your user's password) and create a "digital fingerprint," or hash, of it. Essentially a long string of numbers and letters like this: `$2a$10$vI8aWBnW3fID.ZQ4/zo1G.q1lRps.9cGLcZEiGDMVr5yUP1KUOYTa`. 

Let's get started!

**Step 1** - Add the `gem "bcrypt"` to your Gemfile and run `bundle install` from the terminal.

**Step 2** - Add a `password_hash` column to the users table to store the user's encrypted password.

  + Run this command in your terminal to create a migration to modify your users table:
  ```ruby
  rake db:create_migration NAME=add_password_hash_to_users
  ```
  
  * In the timestamped migration file (under `db/migrate`) replace the `def change` method with these up and down methods
  ```ruby
    def up
      add_column :users, :password_hash, :string
    end

    def down
      remove_column :users, :password_hash, :string
    end
  ```
  * In the terminal run `rake db:migrate`

  * Check your `schema.rb` file (in the `db` directory). You should see a `t.string "password_hash"` in your users table.

**Step 3** - Add the Bcrypt password methods to your User model in `app/models/User.rb` so that your User class looks like this:

```ruby
class User < ActiveRecord::Base
  has_many  :tweets

  include BCrypt

  def password
    @password = Password.new(password_hash)
  end

  def password=(new_password)
    @password = Password.create(new_password)
    self.password_hash = @password
  end
end
```
  + The `include Bcrypt` adds the Bcrypt gem's functionality to your User class.

  + The `def password=` method creates a new password hash for the user - we'll use this to set the user’s password when they sign up.
  
  + The `def password` method will be used to compare the user's saved password to the password they use when logging in. 

That's it for the M in our MVC - now onto the V.

**Step 4** - Add password input fields to the user Sign Up and Sign In forms in your views. 

Finally, onto the C. 

**Step 5** - In your application controller (`app/controllers/application_controller.rb`) go to the `post ‘/sign-up’` route and add the `def password=` method to set the user's password. Your `post ‘/sign-up’` route should now look like this:
```ruby
  post '/sign-up' do
    @user = User.new(:name => params[:name], :email => params[:email])
    @user.password = params[:password]
    @user.save
    session[:user_id] = @user.id
    redirect '/tweets'
  end
```

**Step 6** - In the `‘post ‘/sign-in’` route use the `def password` method to confirm that the user has submitted the correct password. Your `post ‘/sign-in’` route should now look like this:
```ruby
  post '/sign-in' do
    @user = User.find_by(:email => params[:email])
    if @user.password == params[:password]
      session[:user_id] = @user.id
    end
    redirect "/tweets"
  end
```

You are ready to test out your app! 

**If you already have test users in your database you will need to set their passwords via the console - run `tux` in your terminal - before you can test out the sign in form. Or you can delete all your entries by dropping your database - do `rake db:rollback` four times (once for every migration) - and then rake db:migrate to rebuild.** 





<p data-visibility='hidden'>View <a href='https://learn.co/lessons/hs-bcrypt-set-up' title='Setting Up Your App with Bcrypt'>Setting Up Your App with Bcrypt</a> on Learn.co and start learning to code for free.</p>
