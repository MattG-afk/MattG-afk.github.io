---
layout: post
title:      "Sinatra Project Blog"
date:       2020-06-22 16:29:34 +0000
permalink:  sinatra_project_blog
---


Starting off with having a hard time trying to set up my config environment. 

```
ENV['APP_ENV'] ||= "development"

require "bundler/setup"
Bundler.require{:default, ENV['APP_ENV']}

configure :development do
    set :database, (adapter: 'sqlite3', database:'db/db.sqlite3')
end

require_all 'app'

```

I had realized that I messed up my brackets for the adapter section and the bundler.require area.

After fixing that, I was able to run shotgun onto my local host to see if it'll run on a webpage. Once I got to the web page I got the "hello world" good to go sign. 


After I knew the pathing/routing was correct I decided to set a layout and a home erb page. The layout was fairly simple I just did a simple <!DOCTYPE html> that had a welcome message and <%=yield %> so that I could chain my erbs together. 

```
<!DOCTYPE html>
<html lang ="en">
<head>
    <meta charset = "UTF-8">
    <meta name ="viewpoint" content="width-device-width", initial-5>
    <title>Document</title>
</head>
<body>
    <h1>Welcome To Gametopia</h1>
    <%= yield %>
</body>
</html>
```

then had my home.erb set up the Login and Signup with <a href> tags.

```
<h1>Login or Signup to see your games</h1>


<a href="/login">Login</a>
<a href="/signup">Signup</a>
```

So far so good.

I created my models so I could meet the requirements of making sure that the password is secured. I also added a validates for the password and the uniqueness of the email.

```
class User < ActiveRecord::Base
    has_secure_password

    validates :password, :presence => true
    validates :email, :presence => true, :uniqueness => true

    has_many :collections
end
```

After I created my initial erbs I set up my databse and all the tables I would be using I went through and was thinking solely on user and the has-many/belongs-to relation ships that would be going into this project. With video game people can have many but their collection belongs solely on them. So I set up my tables as such. 

At this point, I ran shotgun several times and kept getting small unexpected end errors. Sitting around trying to think what my issue was i realized that my controller didn't have the config/env pathing and my sessions configuration was set up incorrectly so I edited/ fixed it.


```
require "pry"
require "./config/env"
class ApplicationController < Sinatra::Base
    set :views, ->{File.join(root, '../views')}

    configure do
        enable :sessions
        set :session_secret, "sphinx"
    end

    get '/' do
        erb :home
    end


    get '/signup' do
        erb :signup
    end

    post "/signup" do
       @user = User.new(params)
       binding.pry
       if @user.save
        session[:user_id] = @user.id
            redirect "/login"
       else 
           redirect "/failure"
       end
    end

    get "/login" do
        erb:login
    end

    post "/login" do
        #binding.pry
        @user = User.find_by(username: params[:username])
        #binding.pry
        if @user && @user.authenticate(params[:password])
            session[:id] = @user.id 
            redirect "/dashboard"
        else
            redirect "/failure"
        end
    end

    get "/dashboard" do
        if logged_in?
        erb :"dashboard"
        else
        redirect "/login"
        end
    end

    get "/failure" do
        erb :failure 
    end

    get "/logout" do
        session.clear
        redirect "/"
    end

    helpers do
        def logged_in?
            !!session[:user_id]
        end

        def current_user
            User.find(session[:user_id])
        end
    end

end
```


I then ran into the errors of not being able to get my user tables. even though I created them. I realized that i ran migrate earlier and had a blank schema. I ran rake db:drop to clear it and then re-ran rake db migrate to add all my tables. 

```
ActiveRecord::Schema.define(version: 2020_06_20_222324) do

  create_table "collections", force: :cascade do |t|
    t.string "name"
    t.string "console"
    t.string "genre"
    t.integer "user_id"
  end

  create_table "users", force: :cascade do |t|
    t.string "username"
    t.string "email"
    t.string "password_digest"
  end

end

```

Looking into a better user experience I decided to trim my app controller and make a user controller and collection controller.  This way its more organized in my head and routes correspond with that allocated controller. 

#application controller
```
require "./config/env"

class ApplicationController < Sinatra::Base

    configure do
        set :public_folder, 'public'
        set :views, 'app/views'
        enable :sessions
        set :session_secret, "sphinx"
    end

    get '/' do
        erb :'/home'
    end

    helpers do
        def logged_in?
            !!session[:user_id]
        end

        def current_user
           @current_user = User.find_by(session[:user_id])
        end

        def authorized_to_edit?(collection)
            @collection.user == @current_user
        end

    end


end
```

#User Controller

```
class UserController < ApplicationController


    get '/signup' do
        erb :signup
    end

    post "/signup" do
        @user = User.new(params)
        if @user.save
         session[:user_id] = @user.id
             redirect "/login"
        else 
            redirect "/failure"
        end
     end

    get "/login" do
        erb:login
    end

    post "/login" do
        #binding.pry
        @user = User.find_by(email: params[:email])
        #binding.pry
        if @user && @user.authenticate(params[:password])
            session[:id] = @user.id 
            erb :dashboard
        else
            redirect "/failure"
        end
    end


    get "/failure" do
        erb :failure 
    end

    post "/users" do
        if params[:name] != "" && params[:email] != "" && params[:password] != ""
            @user = User.create(params)
            session[:id] = @user.id 
            redirect "/user/#{@user.id}"
        else
            redirect "/failure"
        end
        
    end

    get '/users/:id' do
        @user = User.find_by(id: params[:id])
        erb :'/users/dashboard'
    end

    get "/logout" do
        session.clear
        redirect "/"
    end

end

```


#collection controller

```
class CollectionsController < ApplicationController


    get '/collections' do
        @collection = Collection.all
        erb :'collections/index'
    end

    get '/collections/new' do
        erb :'/collections/new'
    end

    post '/collections/new' do
         if !logged_in?
             redirect '/'
         end

         if params[:name] != "" && params[:genre] != "" && params[:console] != ""
            @collection = Collection.create(params)
            redirect "/collections/#{@collection.id}"
         else
             redirect '/collections/new'
         end
    end

    get '/collections/:id' do
        set_collection
        erb :'/collections/show'
    end

    get '/collections/:id/edit' do
        set_collection
        if logged_in?
            if set_collection.user == @current_user
                erb :'/collections/edit'
            else
                redirect "users/#{current_user.id}"
            end 
        else
            redirect '/'
        end

    end

    patch '/collections/:id' do
        set_collection
        if logged_in?
                if @collection.user == @current_user
                    @collection.update(name: params[:name], genre: params[:genre], console: params[:console])
                    redirect "/collections/#{@collection.id}"
                else
                    redirect "users/#{current_user.id}"
                end
        else
            redirect '/'
        end
    end

    delete'/collections/:id' do
        set_collection
        if authorized_to_edit?(@collection)
            @collection.destroy
            redirect '/collections'
        else
            redirect '/collections'
        end
    end

    private

    def set_collection
        @collection = Collection.find(params[:id])
    end
end
```

All the fuctions in the collection controller are geared towards the collection/ views and also meant for CRUD (create, Read, Update, Delete). I also created a private helper method that sets up a instance variable that finds the collection but id.

After that I pretty much bounce back and forth until I had all my views pages set up and formed correctly.

Running shotgun I found out everything was running correct and that the logic is all there.  

If you would like to check out all of the code or fork the repository feel free to visit the url below.



https://github.com/MattG-afk/gametopia



