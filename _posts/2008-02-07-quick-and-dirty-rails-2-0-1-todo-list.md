---
layout: post
title: Quick and dirty(?) Rails 2.0.1 Todo List
date: '2008-02-07T16:39:00-05:00'
tags:
- Code
- rails
- css
- ruby
- todo
tumblr_url: http://www.hisham.cc/post/30203526420/quick-and-dirty-rails-2-0-1-todo-list
---
So I really needed to get a todo list up and running VERY quickly with minimal hassle. Rails to the rescue! With 2.0.1, you can do the following:



rails todos
cd todo
rake db:create:all
script/generate scaffold Todo title:string body:text done:boolean due:datetime percentage:integer 
rake db:migrate



And then, if you want, add a bit of CSS like I did to create that progress bar, and voila! Insta-TODO. I plan to make the CSS and any other modifications I add to this little Rails-one-liner public very soon (as an update to this post).



