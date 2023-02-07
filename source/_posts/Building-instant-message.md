---
title: Building instant message  
date: 2022-08-16 12:57:55
tags:
---

with Rails6 and Action cable
---


We are going to build a direct message with Ruby on Rails with Action Cable. Action Cable is a framework that allows you to integrate Websocket for Rails, it follows the pub/sub pattern.


---

### Set configs
- Go to `config/cable.yml` 
set async to `redis`, add url `redis://localhost:6379/1.`
- Go to `config/application.rb`, 
add `config.action_cable.mount_path = '/cable'`
- Go to `app/assets/javascripts/application.js` import `cable.js`


---
## Model's parent 
>- in `Application_controller.rb` we define `current_user` with session
>- `current_user` is used in Room's index when we show the current_user 
>- also used in message controller when we create a message and we will know which user it's from.

## Models
 > Room
 > -
>- one room has_many messages
>- in room `index.html.erb`, it shows the current_room, current_consumer and shows the messages.
>- in room `room_controller.rb`, the show action will render :index, so all rooms can be rendered on index 

```shell
$ rails g scaffold Room name
```
 > User
 > -
 >- one user has_many messages
 >- in `user.rb`, it has the code of how the username is generated
 
```shell
$ rails g model User username
```
 > Message
 > -
 >- `messages_contoller`'s action create will link to job `SendMessageJob.perform_later(@message)`
 

```shell
$ rails g scaffold Message content:text room:references user:references
```
#### 


## Channel
>- `room_channel.rb`, add function `subscribed` :`stream_from` "room_channel". `unsubscribed`, `speak(data)`:`ActionCable.server.broadcast`
>- `room_channel.js`, `create.subscription` add all the js that will retrieve message with `connected`, `disconnected`, `received(data)`
>- `connection.rb` is for authenticating consumers
>- `consumer.js` is for connecting consumer

```shell
$ rails g channel room
```

## Job
> in `send_message_job.rb`, it calls `ActionCable.server.broadcast` to the room

```shell
$ rails g job send_message
```


---


### Code
> Go to `Room`'s index view `index.html.erb`
> - we let all the messages show on this page if the room exist
```erb=
<%if @room.present? %>
  <p><%= @room.name%></p>
    <% @room.messages.each do |message| %>
      <%= message.content %>
    <%end%>
<%= render 'messages/form', message: Message.new, room: @room %>
<%end%>
```
> Go to `Room`'s `Model`
```ruby=
  has_many :messages
```
> Go to `Messege`'s `_form .html.erb`
> - we let the message only show the entry boc and subimt, we put room_id hidden_field so we can know which room the message belongs to

**delete**
```erb=
 <div class="field">
    <%= form.label :room_id %>
    <%= form.text_field :room_id %>
 </div>

 <div class="field">
    <%= form.label :user_id %>
    <%= form.text_field :user_id %>
 </div>
```
**add**
```ruby=
<%= form.hidden_field :room_id, value: room.id %>
```

> Go to `Message`'s controller, add below @message declaration
```ruby=
@message.user = current_user 
```

> Go to ApplicationController, adding `helper_method` for `current_user` to make it availble everywhere.
> add a function `current_user`, if we have a current_user id in the session, we're using the user, otherwise we're creating one user.
```ruby!=
helper_method :current_user

def current_user
    return @current_user if @current_user.present?

    if session[:user_id].present?
      @current_user = User.find(session[:user_id])
    else
      @current_user = User.generate
      session[:user_id] = @current_user.id
    end
  end
```

>Go to `User`'s Model `user.rb`, we'll make a class method `generate` which generates usernames:)
```ruby=
def self.generate
    username = "#{["a", "z"].sample}_#{[1, 10].sample}"
    create(username: username)
  end
```

> Go to `Room`’s index view `index.html.erb`
>- show all the rooms options you can go to 

```erb=
<% @rooms.each do |room| %>
      <%= link_to room do %>
        <div><%= room.name %></div>
      <% end%>
    <% end %>
```

> Go to `Message`’s controller, add below in function `create`
>- let message do it's thing and broadcast it to room.
```ruby= 
def create
    @message = Message.new(message_params)
    @message.user = current_user
    @message.save
    ActionCable.server.broadcast "room_channel_#{@message.room_id}", message: message
end

```



>Go to terminal, create a `channel` named [room]
```shell=
rails g channel room
```
![](https://i.imgur.com/d7bUr2v.png)

>Go to channels file tree, select the`room_channel.rb`, add the `stream_from` 

```ruby=
def subscribed
    stream_from "room_channel_#{params[:room_id]}"
  end
```

> Go to `room_channel.js`
```javascript=
consumer.subscriptions.create({ channel: "RoomChannel", room_id: 1}, {
  connected() {
    console.log("connected!");
  },
    
  disconnected() {
    
  },

  received(data) {
    console.log(data);
  }
});
```

> Go to `Message`’s controller, add below in function `create`
```ruby=
def create
    @message = Message.new(message_params)
    @message.user = current_user
    @message.save
    
    html = render(
        partial: 'messages/message',
        locals: {message: @message}
    )
    
    ActionCable.server.broadcast "room_channel_#{@message.room_id}", html: html
end

```

> Go to `messages` filetree, and add a file `_message.html.erb`, which is the `partial` in 
```ruby=
<%= 'me' if message.user == current_user%%>
      <%= message.content %>
            <%= message.user.username %>

```

> Go to `Room`'s index view `index.html.erb`, add an id so later we could go to `room_channel.js` to get the message.
```erb
<%if @room.present? %>
  <p><%= @room.name%></p>
    <div id = "messages">
    <% @room.messages.each do |message| %>
        <%= render 'messages/message', message: message %>
    <%end %>
    </div>
<%= render 'messages/form', message: Message.new, room: @room %>
<%end%>
```

> Go to terminal, generate a `job` called [send_message]
```shell=
$ rails g job send_message
```

> Go to `send_message_job.rb`, remove the html and ActionCable code from `message_controller` to here, and add `ApplicationController`, remove the @ from messages as instance variables.
```ruby=
 def perform(message)
    html = ApplicationController.render(
      partial: 'messages/message',
      locals: {message: message}
  )
  
  ActionCable.server.broadcast "room_channel_#{message.room_id}", html: html
  end
```

> Go back to `message_controller`, add below in function create
```ruby=
def create
    @message = Message.new(message_params)
    @message.user = current_user
    @message.save
    SendMessageJob.perform_later(@message)
  end
```

>Rerun rails s

>Go to `room_channel.js`
```javascript=
 received(data) {
    console.log(data);
    const = messageContainer = document.getElementById('messages')
    messageContainer.innerHTML = messageContainer.innerHTML + data.html
  }
```

> Go to `send_message_job.rb`, change html into my_message and their_message
```ruby=
 class SendMessageJob < ApplicationJob
  queue_as :default

  def perform(message)
    my_message = ApplicationController.render(
      partial: 'messages/my_message',
      locals: {message: message}
  )

  their_message = ApplicationController.render(
    partial: 'messages/their_message',
    locals: {message: message}
)
  
  ActionCable.server.broadcast "room_channel_#{message.room_id}", my_message: my_message, their_message: their_message, message: message
  end
end

```

>In the messages file, create `_my_message.html.erb`, and `_their_message.html.erb`
```ruby=
<%= 'me' if message.user == current_user%%>
  <%= message.content %>
      <%= message.user.username %>
```
```ruby=
 <%= message.content %>
      <%= message.user.username %>
```

>Go to `room_channel.js`, add element and user-id.
```javascript=
 received(data) {
    console.log(data);
    const element = document.getElementById('user-id')
    const user_id = Number(element.getAttribute('data-user-id'))
    
    let html
    if(user_id === data.message.user_id){
        html = data.my_message
    }else{
        html = data.their_message
    }
     const = messageContainer = document.getElementById('messages')
    messageContainer.innerHTML = messageContainer.innerHTML + data.html
  }

```

>Go to `index.html.erb`, add these on the very top
>- to let `room_channel.js` get its DOM
```ruby=
<div id = "room_id" data-room-id ="<%= @room.try(:id) %>"></div>
<div id = "room_id" data-room-id ="<%= current_user.id %>"></div>
```

>Go to `room_channel.js`
```javascript=
import consumer from "./consumer"

document.addEventListener('turbolinks:load', () => {

  const room_element = document.getElementById('room-id');
  const room_id = room_element.getAttribute('data-room-id');

  console.log(room_id)

  consumer.subscriptions.create({ channel: "RoomChannel", room_id: room_id }, {
    connected() {
      console.log("connected to " + room_id)
      // Called when the subscription is ready for use on the server
    },

    disconnected() {
      // Called when the subscription has been terminated by the server
    },

    received(data) {
      const user_element = document.getElementById('user-id');
      const user_id = Number(user_element.getAttribute('data-user-id'));

      let html;

      if (user_id === data.message.user_id) {
        html = data.my_message
      } else {
        html = data.their_message
      }

      const messageContainer = document.getElementById('messages')
      messageContainer.innerHTML = messageContainer.innerHTML + html
    }
  });
})
```
 A fully functional chatroom, yay!