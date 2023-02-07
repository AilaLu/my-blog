---
title: Action Cable Rails 6.1.6.1
date: 2022-08-23 10:37:59
tags:
---

## Connections
#### a subscriber per browser tab
- Connection is an `object` for every `Websocket` that is accepted from the server end.
- This connection `object` is the parent of all `channel subscriptions. 
- This connection `object` is also the instance of the module `ApplicationCable::Connection`
- Connection doesn't include authentication of the subscribers. 
```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private
      def find_verified_user
        if verified_user = User.find_by(id: cookies.encrypted[:user_id])
          verified_user
        else
          reject_unauthorized_connection
        end
      end
  end
end
```

## Channels
#### a `channel` is similar to a `controller` in rails, which contains all the logic.
- Server: Consumers can subscribe to multiple channels by function `subscribed` in `room_channel.rb`
- Consumer: The `subscription` occurs when consumer.subscriptions.create to channel in `room_channel.js`

```shell
$rails g channel [channel name]
```
if we generated a channel room
```shell
$rails g channel room
```
it will automatically generated a `room_channel.rb` and `room_channel.js`
>in `room_channel.rb`, we have `streams` that broadcasts the data to subscribers.
```ruby 
class RoomChannel < ApplicationCable::Channel
  def subscribed
    stream_from "room_channel_#{params[:room_id]}"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end
```
>in `room_channel.js`

```javascript
consumer.subscriptions.create({ channel: "RoomChannel", room_id: room_id}, {
  connected() {
    console.log("connected!");// Called when the subscription is ready for use on the server
  },

  disconnected() {
    // Called when the subscription has been terminated by the server
  },

  received(data) {
    this.appendLine(data)
  },

  appendLine(data) {
    const html = this.createLine(data)
    const element = document.querySelector("[data-chat-room='Best Room']")
    element.insertAdjacentHTML("beforeend", html)
  },

  createLine(data) {
    return `
      <article class="chat-line">
        <span class="speaker">${data["sent_by"]}</span>
        <span class="body">${data["body"]}</span>
      </article>
    `
  }
});
```

