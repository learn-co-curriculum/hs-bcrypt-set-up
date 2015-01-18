---
tags: cryptography, bcrypt, kids, ruby, advanced, study guide
language: ruby
level: 2
type: study guide
---

## Setting Up Your App with Bcrypt

We want our users to be able to sign in with a password but we can't just save user passwords in plain text. That makes it too easy for someone to hack in to our system and steal everyone's secrets.

We’re going to set up secure password storage with a gem called bcrypt that will encrypt our users passwords before we save them in our database. 
  * You can learn more about bcrypt [here](https://github.com/codahale/bcrypt-ruby).

+ Bcrypt() is a hashing algorithm, which means it takes a chunk of data (e.g., your user's password) and create a "digital fingerprint," or hash, of it.  * Essentially a long string of numbers and letters like this: $2a$10$vI8aWBnW3fID.ZQ4/zo1G.q1lRps.9cGLcZEiGDMVr5yUP1KUOYTa. 
+ Our first step is to add the bcrypt gem to our Gemfile
  ```ruby
     gem "bcrypt"
  ```
  and run `bundle install` from our terminal.
+ Next we'll need to add a `password_hash` column to our users table so we can store our user's encrypted password. 
  * Step 1 - Create a migration to modify your users table:
  ```ruby
  rake db:create_migration NAME=”add_password_hash_to_users”
  ```
  which will create a new timestamped migration file in `db/migrate`
  * Step 2 - In the timestamped migration file replace the `def change` method with these up and down methods
  ```ruby
    def up
      add_column :users, :password_hash, :string
    end

    def down
      remove_column :users, :password_hash, :string
    end
    ```
  * Step 3 - In your terminal run `rake db:migrate`
  * Step 4 - Check your `schema.rb` file (in the db directory). You should see a `t.string "password_hash"` in your users table.
+ Next we'll need to add some standard code to our users model in `User.rb` (in the app/models directory) so that your User class looks like this:

```ruby
class User < ActiveRecord::Base
  has_many  :tweets

  include BCrypt

  def password
    @password ||= Password.new(password_hash)
  end

  def password=(new_password)
    @password = Password.create(new_password)
    self.password_hash = @password
  end
end
```

Great! Now let’s take a look at the rest of this example code in the bcrypt documentation.  
Line 5 - We haven’t talked about this in class but there is something in Ruby called a Module. Modules are used to help keep our code organized and make it easier to reuse and share.
If we dig into the bcrypt gem we’ll find BCrypt module that contains a Password class with a create method. 
Line 7 down should look a little familiar.
It’s been a while since we’ve created readers and writers, but that is essentially what these methods are. 
The `def password=` method creates a new password hash for the user
This is a method that we would use to set the user’s password when they sign up for an account OR if we add functionality to allow users to reset their password (which we should).
The reader method `def password` decrypts the saved password_hash Password.new(password_hash) so we can compare it to the password that the user submits in the log in form.
This `||=` is something we haven’t seen before. It’s essentially means if @password is not already set then set it to the decrypted password.
I think bcrypt does this because you might have an app with several different entry points and if it has already run the decryption once during the session then @password has already been set and you don’t need to run Password.new again.
Remember running the decryption is intentionally slow and expensive so you don’t want to run it anymore than you have to.
You don’t need to worry about this though for our little app (so if it’s confusing just delete the ||) and understand that calling  @user.password will return the user’s decrypted password_hash.
Let’s go ahead and copy and paste this code from `include BCrypt` on down to the end of the password= method and paste it into our User model.
So now we have our Model portion of our application set up for creating and storing passwords but don’t forget every time we add functionality to our website we need to think about all the parts of the MVC. So what comes next?
The views. We’ll need to take in a password from our users when they sign up and sign in. What do we need to modify? Everyone add a password input field to their sign up and sign in forms. 
Finally, the controller. This is where we’ll be writing the code to actually use those password and password= methods. 
Let’s start with the `post ‘/sign-up’`. The User.new and user.save methods are good we want to keep those. What else needs to happen now that we are also taking in a params[:passwor]? Hint: Don’t overthink it! You know how to use readers and writers to set an attribute and access an attribute. 
We need to set the password, right? Which means using the writer method, password= method and setting @user.password = params[:password]
Don’t even worry about what is happening under the hood! Bcrypt has got you covered. 
Now let’s move on the ‘post ‘/sign-in’ route. Here we are looking for the user and if they exist we give them a session id. Now we need to confirm not only that they exist but that their password matches. How do we access a user’s password? With the reader `def password`. What do we want to compare this to? The password that the user has just input params[:password]. So `if @user.password == params[:password]` create a session[:user_id]

You actually don’t need to remember anything we just talked about. As long as you use the == for comparison, just like you normally would you can check if a stored password_hash is equal to a submitted password (@user.password == submitted_password).
