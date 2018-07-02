---
layout: post
title:      "Collect Nation - Sinatra CRUD MVC"
date:       2018-07-02 02:16:40 +0000
permalink:  collect_nation_-_sinatra_crud_mvc
---


This final project took awhile, but it was well worth it! It's the first functional Sinatra CRUD app that I have built from the ground up not as a lab. Having done a bunch of labs that required application of sinatra, activerecord, rake, and sqlite3, I did not feel so intimidated going into this project. I decided to start by writing my own tests using rspec and capybara. I looked at previous sinatra/activerecord labs to make sure I had the right syntax. Figuring out the logic for each test was difficult but not impossible. I made a total of 6 spec.rb files, 1 for each model and 1 for each controller and its views.


The next step was to create my models. The minimum requirements mandated 1 has_many and 1 belongs_to relationship, but I decided to go 1 step further than that. My models are user, collection, and item. A user has many collections and has many items through collections. A collection belongs to a user and has many items. An item belongs to a collection. Setting up the models was no problem - I just had to include a foreign key in both collection and item (:user_id and :collection_id, respectively).

Creating the table migrations was the next step. Users have a username, email, and password (made secure with password_digest, bcrpyt gem, and has_secure_password in the user model). Collections have a name, description, and the user_id foreign key. Items have a name, description, and the collection_id foreign key. After setting up the migrations, I ran rake db:migrate and rake db:migrate SINATRA_ENV=test to establish my database for my server and my database for my test suite. I also created a seed file with model data so that my website would already have a couple users, collections, and items already set up. I ran the 3 model specs and quickly got each one passing.

The next step was to begin going thru my controller_views specs and adding all the routes in the controllers as well as building each view. This was by far the most daunting part of the whole process - I had to set up my website so that users could only create/read/update/delete their own collections and items, not another user's, but still be able to view data from other users. This involved a lot of time defining if/else statements and running shotgun to make sure the logic worked for both the test suite and my actual server. I may have spent close to an hour or more on a few key methods to ensure that they worked. The most difficult methods were the GET /collections/new and PATCH /collections routes. The former was difficult because it was responsible for not only creating a new collection, but also creating a new item that belongs to that newly created collection. The latter was difficult because clicking the edit button that leads to the collection form then allows the user to click an edit item button that leads to the item's own edit page! I had to rewrite a bunch of specs so that my test suite would pass and my website would work simultaneously.

Overall, the process of building my website from start to finish was longer and tougher than I initially predicted, but it was still rewarding just the same. As for css styling, I will probably go back and implement it later - I'm not really into design and don't see myself doing only front-end development in the future. Stay tuned for more updates - it's only going to become more challenging from here on out!
