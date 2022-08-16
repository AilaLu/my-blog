---
title: find and find_by
date: 2022-08-16 00:44:23
tags:
---


  `Find` and `find_by` are the methods that are super common in writing Ruby on Rails CRUD, especially in the controller actions `show` `edit` `update` `destroy`.
  What we try to do with these methods is to find the according element based on their indices or attributes, meaning the columns in your model!

  Now let’s see some examples to understand their differences:
## find
```rb
Book.find(params[:id]) #find book id
Article.find(params[:article_id]) #find article id in comment controller
```
## find_by
```rb
Book.find_by(id: params[:id]) #find book id
Book.find_by(title: params[:title]) #find book title 
User.find_by(id: session[:user_session]) #find the current user
```
  With `find`, it can only access the id attribute, and return error when the element is not found. Whereas with `find_by`, you can access to any attribute you like, as we see with the `book title`.
  One thing to notice with find_by is that it returns nil when the element is not found, which left you with not much clue of what went wrong. Thus, you can add ! to your code, and it will return error instead.
```rb
Book.find_by!(id: params[:id]) #returning error with find_by
```
