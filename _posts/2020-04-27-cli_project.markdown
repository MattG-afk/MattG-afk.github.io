---
layout: post
title:      "Cli Project"
date:       2020-04-27 17:15:26 -0400
permalink:  cli_project
---


Starting this project I felt like i had no idea what direction I wanted to go with. After some time and consideration I decided to go with a gem that would be something that interested me, mainly music. So I began the process with the idea that I wanted this gem to specifically find the top 100 Dubstep artists. The main issue was a lot of the site I checked(ie. Ranker, Dj Mag, Etc.) didn't really have the songs that made that artist popular, but after think about having the songs included I decided to go to beatport. On beatport, it had all the available info that i would need to scrape of and put into my gem. I began with creating a cli, trends, and scraper ruby file. After creating my instance method in my trends file.

I wanted to make all the class methods for it so I decided to just start with the clear and all methods until I built off my other files. I knew after creating those class methods I would need to establish my cli and scraper files. So I decided to do a small amount of work in the cli project just starting it. I also establish my scraper class only with one class method that would be able to get pulled into my trends file. At this point, it was all testing and trying to figure out what class methods would be able to find the Artist and the song that they had. The last main thing I needed to do was add easy fuctionality to my cli. What I did was I had the program wait for user input and as a user input a valid response it would show them the Artist. After they select a artist, it would show the song and the ranking of that song. If the user ever types exit it would exit the program completely.
