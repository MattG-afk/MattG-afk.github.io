---
layout: post
title:      "Rails Project #2"
date:       2020-12-11 22:58:11 -0500
permalink:  rails_project
---


When I originally did the rails project. I failed due to several issues with my routing and not meeting the specific requirements. This time around in my blogpost I'm going to make sure that I meet the requirements by making a checklist with the examples. So lets get to it.


**#1 • Include at least one has_many, at least one belongs_to, and at least two has_many :through relationships

• Include a many-to-many relationship implemented with has_many :through associations. The join table must include a user-submittable attribute — that is to say, some attribute other than its foreign keys that can be submitted by the app's user**

This was the most difficult in terms of relationships when I did my project the first time around. I kept overcomplicating things. This time around, I did a project that made sense of the relationships. Video Games. 

```
class User < ApplicationRecord

 has_many :games
 has_many :comments, through: :games
 
 end

```

```
class Game < ApplicationRecord

    has_many :comments
    has_many :users, through: :comments


 end

```

```
class Comment < ApplicationRecord

    belongs_to :user
    belongs_to :game 


end

```

With this, the relatioship of User and Games will be connected through comments so that a User can have many games and vise vera.

**#2Your models must include reasonable validations for the simple attributes. You don't need to add every possible validation or duplicates, such as presence and a minimum length, but the models should defend against invalid data.**

In terms of validations, it was pretty easy due to the fact that the method is already in rails and I can easily apply that to my models like so:

```
class Game < ApplicationRecord
    validates :name, presence: true  
    validates :genres, presence: true
    validates :name, length: {maximum: 250}
    validates :description, length: {maximum:2000}
end
```

```

class User < ApplicationRecord
         validates :username, presence: true, uniqueness: true
         validates_presence_of   :email, presence: true, uniqueness: true, :message => 'Please Enter your Email.'
    validates_presence_of   :password, presence: true, uniqueness: true, :message => 'Please Enter Your Password.'
        has_secure_password

end
```


```
class Comment < ApplicationRecord
   validates :content, length: {maximum: 250} 
    validates :stars, numericality: {only_integer: true, greater_than_or_equal_to: 0, less_than: 6}

end
```

**#3You must include at least one class level ActiveRecord scope method. a. Your scope method must be chainable, meaning that you must use ActiveRecord Query methods within it (such as .where and .order) rather than native ruby methods (such as #find_all or #sort). **


With this I had to research what would be the best option for my users. I decided to have a search bar funtion where they could find the game they created. 



```
Class Game < ApplicationRecord

scope :search_by_name, -> (search_name){where("name = ?", search_name)}

end

```

I then added it to my index.html.erb page for games and to my index method in my Games Controller as well.

**#4Your application must provide standard user authentication, including signup, login, logout, and passwords.**

**#5 Your authentication system must also allow login from some other service. Facebook, Twitter, Foursquare, Github, etc...,**

**#7 Your forms should correctly display validation errors.
a. Your fields should be enclosed within a fields_with_errors class
b. Error messages describing the validation failures must be present within the view.**

This was really simple because of our previous lessions that show you how to set this all up in the past sinatra module. 

With the Omniauth, l had some issues with it correctly authenticating and routing the user into the games index. I then relized that I needed to redirect correctly in my omniauth method. 

I found that with the field with errors I had to manually create a errors page so that when a user leaves the fields blank it will correctly bring up the error partial. 


```
    validates :username, presence: true, uniqueness: true
    validates_presence_of   :email, presence: true, uniqueness: true, :message => 'Please Enter your Email.'
    validates_presence_of   :password, presence: true, uniqueness: true, :message => 'Please Enter Your Password.'
    has_secure_password
```

User new.html.erb
```
<h1>Register for GameGo</h1>

<div class = "fields_with_errors"><%= render 'layouts/user_login_error' %></div>

<%=form_for(@user, url: "/signup") do |f| %>
    <p>
    <%= f.label "Username:"%>
    <%= f.text_field "username"%>
    <%= f.label "Email:"%>   
    <%= f.text_field "email"%>
    <%= f.label "Password:"%>
    <%= f.text_field "password"%>
    <%= f.label "First Name:"%>
    <%= f.text_field "first_name"%>     
    <%= f.label "Last Name:"%>
    <%= f.text_field "last_name"%>
    </p>
    <%= f.submit "Register"%>
<% end %>

```

Sessions new.html.erb
```
<%=form_tag("/sessions") do %>
    <p>
    <%= label_tag "Email:"%>   
    <%= text_field_tag "email"%>
    <%= label_tag "Password:"%>
    <%= text_field_tag "password"%>
    </p>
    <%= submit_tag "Login"%>
<% end %>

<%= link_to "Log In with Google", '/auth/google_oauth2' %>
```
Layouts application.html.erb
```
<!DOCTYPE html>
<html>
  <head>
    <title>GameGo</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
      <div class="navbar">
      <nav>
     <div class="logo">
GameGo</div>
<ul class="menu-area">


  <% if logged_in? %>
  <li><a href="/">Home</a></li>
   <li><%= link_to "Games", games_path %></li>
  <li><%= link_to "Create Game", new_game_path %></li>
   <li><%= link_to "Logout", logout_path, method:"delete" %></li>
  <% else %>
  <li><a href="/signup">Signup</a></li>
  <li><a href="/login">Login</a></li>
  <% end %>
</ul>
</nav>    
</div>
    <%= yield %>
  </body>
</html>
```

The methods for these are in the Users and Sessions controllers.


**#6 You must include and make use of a nested resource with the appropriate RESTful URLs.
• You must include a nested new route with form that relates to the parent resource
• You must include a nested index or show route**

Nesting when I did my first project were still a little fuzzy in terms of understanding what they do, but after redoing the course, I found out that it helps make routing easier. 

Instead of having to type out every single route for comments for the games instead I can just nest route it!

Example

```
get 'games/:id/comments', to: 'games#comments_index'
	
	
	INSTEAD
	
	resources :games do
	   resources :comments, only: [:index]
	end
```

**#8 Your application must be, within reason, a DRY (Do-Not-Repeat-Yourself) rails app. **

This is fairly easy, all you have to do is understand if you are writing the same code over and over again.
Just make a method that will allow you too do that same function over and over, then impliment it in your previous methods. I mainly kept repeating functions in my create, edit and update functions so I made methods that would allow me to write shorter cleaner code. 

```
 private

    def find_game 
        @game = Game.find_by(id: params[:id])
    end

    def game_params
        params.require(:game).permit(:name, :genres, :description,comments_attributes:[:stars,:content,:user_id])
    end
end

```

```
 def edit
        find_game
    end

    def update
        find_game
        if @game.update(game_params)
            redirect_to game_path(@game)
        else
            render :edit
        end
    end
```



In terms of Omniauth, If you need more clarification with it I recommend checcking out Rachel Hawas Blog Post on medium.com.


Thanks For Reading :)

