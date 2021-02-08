---
layout: post
title:      "Javascript + Rails Api Project"
date:       2021-02-08 03:16:52 +0000
permalink:  javascript_rails_api_project
---


   After much time thinking about what I wanted to do for this project, I decided to go with a game. I tested some thing out when it came to what I should do and what would make sense. I realized that the best game for now would be a dodgeball-esque game. Which allowed users to dodge the objects and make it to a safe space. 

So to start out I did my backend which would be the Rails Api. The Api is the server which allows to frontend to talk to the backend and vice versa. When creating the backend I wanted to make it as simple as possible, so I when with a User and Comment Model, which allows requirement #3 of the project pass:

 `User Model:

 validates :username, uniqueness: true, presence: true
	has_many :comments

 Comment Model:

	belongs_to :user
	validates :text, presence: true`
	
	
	For requirement #1 and #4, they require you to make 3 Ajax calls with Crud Actions while everything with the frontend and backend are working at the same time. (this is an abridged version of the requirements.) To allow this to happen I set up my controllers to do crud actions with the users and comments but to render json with them that allows the data being sent from the frontend to be store as data objects in the backend.
	
	` User Controller:
	
	  class UsersController < ApplicationController
    def index
        users = User.all
        render json: users, only: [:id, :username, :levels_completed]
    end

    def show
        user = User.find(params[:id])
        render json: user, only: [:id, :username, :levels_completed]
    end

    def create
        user = User.new(user_params)
        if user.save
            session[:user_id] = user.id 
            render json: user, only: [:id, :username, :levels_completed]
        else
            render json: user.errors.full_messages
        end
    end

    def increase_level
        user = User.find_by(username: params[:username])
        if user 
            user.levels_completed = 1
            user.save
            render json: user, only: [:id, :username, :levels_completed]
        end
    end


    private 

    def user_params
        params.require(:user).permit(:username, :levels_completed)
    end
end

Comments controller: 

class CommentsController < ApplicationController
    def index
        comments = Comment.all 
        render json: comments, include: [:user]
    end
    
    def create
        comment = Comment.new(user_id: params[:user_id], text: params[:comment])
        if comment.save
            render json: comment, include: [:user]
        else
            render json: comment.errors.full_messages
        end
    end
end


Sign up Function:

function signUp(e) {
    e.preventDefault();
    
    let userInputForUsername = document.querySelector("#usernameForSignUp").value;
    
    let formData = {
        username: userInputForUsername
    }
    
    let configObj = {
        method: "POST", 
        headers: {
            "Content-Type": "application/json",
            "Accept": "application/json"
        },
        body: JSON.stringify(formData)
    }
    
    fetch(`${BACKEND_URL}/users`, configObj)
    .then(resp => resp.json())
    .then(parsedResp => {
        if (parsedResp.username) {
            
            newComment(parsedResp.id);

            const currentUser = document.querySelector("#current-user");
            currentUser.innerText = parsedResp.username;
            
            const highScore = document.querySelector("#levels-completed");
            if (parsedResp.levels_completed === 0) {
                highScore.innerText = `${parsedResp.levels_completed} levels completed`;
            } else if (parsedResp.levels_completed === 1) {
                highScore.innerText = `${parsedResp.levels_completed} level completed`;
            }
        }
    });
}`
	
  I do have other functions that enables fetch to manipulate the verb http request and the API as well, but to get to the final requirement.The Javascript application needs to use Object Oriented Javascript(classes) for related behavior and data. For this I used a Player, Enemy, Goal, Game, and a InputHandler class for all the related movement and corresponding data for them.
	
	`Player Javascript Class:
	
	class Player {
    constructor(game) {
        this.width = 30;
        this.height = 30;

        this.maxSpeed = 5;
        this.speed = 0;
        this.axis; 

        this.game = game;
    

        this.position = {
            x: 20,
            y: 20
        }
    }

    draw(context) {
        context.fillStyle = "#00f"
        context.fillRect(this.position.x, this.position.y, this.width, this.height);
    }

    moveLeft() {
        this.speed = -this.maxSpeed;
        this.axis = "x";
    }

    moveRight() {
        this.speed = this.maxSpeed;
        this.axis = "x";
    }

    moveDown() {
        this.speed = this.maxSpeed;
        this.axis = "y";
    }

    moveUp() {
        this.speed = -this.maxSpeed;
        this.axis = "y";
    }

    moveTopRight() {
        this.speed = this.maxSpeed;
        this.axis = "xy1";
    }

    moveTopLeft() {
        this.speed = this.maxSpeed;
        this.axis = "xy2";
    }

    moveDownLeft() {
        this.speed = this.maxSpeed;
        this.axis = "xy3";
    }

    moveDownRight() {
        this.speed = this.maxSpeed;
        this.axis = "xy4";
    }

    stop() {
        this.speed = 0;
    }

    update(deltaTime) {
        if (!deltaTime) return;


        if (this.axis === "y") {
            this.position.y += this.speed;
        } else if (this.axis === "x") {
            this.position.x += this.speed;
        } else if (this.axis === "xy1") {
            this.position.x += this.speed;
            this.position.y += -this.speed;
        } else if (this.axis === "xy2") {
            this.position.x += -this.speed;
            this.position.y += -this.speed;
        } else if (this.axis === "xy3") {
            this.position.x += -this.speed;
            this.position.y += this.speed;
        } else if (this.axis === "xy4") {
            this.position.x += this.speed;
            this.position.y += this.speed;
        }

        
        if (this.position.x < 0) {
            this.position.x = 0;
        }
        if (this.position.x + this.width > 800) {
            this.position.x = 800 - this.width;
        }
        if (this.position.y < 0) {
            this.position.y = 0;
        }
        if (this.position.y + this.height > 600) {
            this.position.y = 600 - this.height;
        }
    }

}

class Enemy {
    constructor(xPosition, game) {
        this.width = 60;
        this.height = 60;

        this.speed = 18;
        this.axis; 

        this.position = {
            x: xPosition,
            y: 0
        }


        this.game = game;
    }

    draw(context) {
        context.fillStyle = "#f00"
        context.fillRect(this.position.x, this.position.y, this.width, this.height);
    }

    update(deltaTime) {
        if (!deltaTime) return;

        this.position.y += this.speed;

        if (this.position.y + this.height > 600) {
            this.speed = -this.speed;
        }
        if (this.position.y < 0) {
            this.speed = 20;
        }

        let rect1 = {
            x: this.game.player.position.x,
            y: this.game.player.position.y, 
            width: this.game.player.width,
            height: this.game.player.height
        }


        let rect2 = {
            x: this.game.enemy1.position.x,
            y: this.game.enemy1.position.y, 
            width: this.game.enemy1.width,
            height: this.game.enemy1.height
        }


        let rect3 = {
            x: this.game.enemy2.position.x,
            y: this.game.enemy2.position.y, 
            width: this.game.enemy2.width,
            height: this.game.enemy2.height
        }


        let rect4 = {
            x: this.game.enemy3.position.x,
            y: this.game.enemy3.position.y, 
            width: this.game.enemy3.width,
            height: this.game.enemy3.height
        }

        let rect5 = {
            x: this.game.goal.position.x,
            y: this.game.goal.position.y, 
            width: this.game.goal.width,
            height: this.game.goal.height
        }


        if (rect1.x < rect2.x + rect2.width &&
            rect1.x + rect1.width > rect2.x &&
            rect1.y < rect2.y + rect2.height &&
            rect1.y + rect1.height > rect2.y) {
            this.game.player.position.x = 20;
            this.game.player.position.y = 20;
        }

        if (rect1.x < rect3.x + rect3.width &&
            rect1.x + rect1.width > rect3.x &&
            rect1.y < rect3.y + rect3.height &&
            rect1.y + rect1.height > rect3.y) {
            this.game.player.position.x = 20;
            this.game.player.position.y = 20;
        }


        if (rect1.x < rect4.x + rect4.width &&
            rect1.x + rect1.width > rect4.x &&
            rect1.y < rect4.y + rect4.height &&
            rect1.y + rect1.height > rect4.y) {
            this.game.player.position.x = 20;
            this.game.player.position.y = 20;
        }

        if (rect1.x < rect5.x + rect5.width &&
            rect1.x + rect1.width > rect5.x &&
            rect1.y < rect5.y + rect5.height &&
            rect1.y + rect1.height > rect5.y) {
            win();
            updateLevelsCompleted();
            this.game.player.stop();
            this.game.player.position.x = 20;
            this.game.player.position.y = 20;
        }
    }
}

lass Goal {
    constructor(game) {
        this.width = 30;
        this.height = 30;

        this.position = {
            x: 780 - this.width,
            y: 580 - this.height
        }
    }

    draw(context) {
        context.fillStyle = "#FFFF00"
        context.fillRect(this.position.x, this.position.y, this.width, this.height);
    }

    update(deltaTime) {
        if (!deltaTime) return;


    }
}

class InputHandler {
    constructor(player) {
        document.addEventListener("keydown", (e) => {
            switch(e.key) {
                case "a":
                    player.moveLeft();
                    break;
                case "w":
                    player.moveUp();
                    break;
                case "d":
                    player.moveRight();
                    break;
                case "s":
                    player.moveDown();
                    break;
                case "p":
                    player.moveTopRight();
                    break;
                case "o":
                    player.moveTopLeft();
                    break;
                case "l":
                    player.moveDownLeft();
                    break;
                case ";":
                    player.moveDownRight();

            }
        })

        document.addEventListener("keyup", (e) => {
            switch(e.key) {
                case "a":
                    player.stop();
                    break;
                case "w":
                    player.stop();
                    break;
                case "d":
                    player.stop();
                    break;
                case "s":
                    player.stop();
                    break;
                case "p":
                    player.stop();
                    break;
                case "o":
                    player.stop();
                    break;
                case "l":
                    player.stop();
                    break;
                case ";":
                    player.stop();
                    break;
            }
        })
    }
}

class Game {
    constructor() {
        this.gameWidth = 800;
        this.gameHeight = 600;


    }

    start() {
        const enemyWidth = 60;
        const enemyHeight = 60;
        let enemyXPosition = 400 - enemyWidth / 2;

        this.player = new Player(this);
        new InputHandler(this.player);
        this.enemy1 = new Enemy(enemyXPosition, this);
        this.enemy2 = new Enemy((200 - enemyWidth / 2), this);
        this.enemy3 = new Enemy((600 - enemyWidth / 2), this);
        this.goal = new Goal(this);
    }

    update(deltaTime) {
        this.player.update(deltaTime);
        this.enemy1.update(deltaTime);
        this.enemy2.update(deltaTime);
        this.enemy3.update(deltaTime);
        this.goal.update(deltaTime);
    }

    draw(context) {
        this.player.draw(context);
        this.enemy1.draw(context);
        this.enemy2.draw(context);
        this.enemy3.draw(context);
        this.goal.draw(context);
    }
}`


With all the requirements set you will have a fully functioning game that will allow you to move your character piece to the safe zone as long as you dodge the somewhat fast enemy pieces. 

