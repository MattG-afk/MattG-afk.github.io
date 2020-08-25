---
layout: post
title:      "Rails Project"
date:       2020-08-24 23:40:53 -0400
permalink:  rails_project
---


Starting out in my rails project, I wanted to create a application that could save recipes to a users account, so they don't have to worry about if they lost old recipe books or written down family recipes. 

The first thing I wanted to start with was the models and associations so that I could have everything that needed to be related in my DB set up and that the logic for it was there

```
User Model

class User < ApplicationRecord
    validates_presence_of   :email, :message => 'Please Enter User  Name.'
    validates_presence_of   :password, :message => 'Please Enter Your Password.'
    has_secure_password
    has_many :ingredients
    has_many :recipes, through: :ingredients   

        def self.from_omniauth(auth)
        where(email: auth.info.email).first_or_initialize do |user|
          user.username = auth.info.name
          user.email = auth.info.email
          user.password = SecureRandom.hex
        end
    end

end

Recipe Model

class Recipe < ApplicationRecord
        has_many :ingredients
        has_many :users, through: :ingredients
        accepts_nested_attributes_for :ingredients
end

Ingredient Model

class Ingredient < ApplicationRecord
    belongs_to :recipe
    belongs_to :user
end


```

My DB was set up accordingly as well 

```
ActiveRecord::Schema.define(version: 2020_08_17_165601) do

  create_table "ingredients", force: :cascade do |t|
    t.string "name"
    t.string "quantity"
    t.integer "user_id"
    t.integer "recipe_id"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "recipes", force: :cascade do |t|
    t.integer "recipe_id"
    t.string "name"
    t.string "content"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "users", force: :cascade do |t|
    t.string "username"
    t.string "email"
    t.string "password_digest"
    t.string "first_name"
    t.string "last_name"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

end

```

The biggest issue when it came to my project was showing the correct attributes in the html and tweaking the controllers to do the actions that I wanted them to do. Especially my Omniauth method...

At first, when I set it up the errors I got were incorrect class id but I fixed that with recreating the Keys I got from google. From that point, everything worked out on that end.

After that I had a issue with showing all my recipes on the index page I had asked another peer what initially might be going on but after spending a good hour on it I finally was able to find the issue of it not being routed correctly through my routes.rb file. I adjusted and slightly tweaked how the route would present the page and it worked. The main tool used for that was binding.pry just to make sure all the strong params were still available in the @recipe instance variable. 

```
 def new
      @recipe = Recipe.new
      @recipe.ingredients.build(name: "name")
      @recipe.ingredients.build(quantity: "quantity")

    end
    
    def create
      @recipe = Recipe.create(recipe_params)
  
      @recipe.save

      #binding.pry
      redirect_to recipes_path
    end
    
      private
        def recipe_params
          params.require(:recipe).permit(:name,:content, ingredients_attributes: [
            :recipe_id,
            :user_id,
            :name,
            :quantity
            ]
          )
      end


```

If you would like to check out my code feel free to visit [] https://github.com/MattG-afk/Formuoli-app(http://). 


