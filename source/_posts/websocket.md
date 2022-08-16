---
title: websocket
date: 2022-08-16 11:51:40
tags:
---

`Websocket` is a protocol that allows us to have `bi-directional communication`, you can think of it as an open door between `client` and `server`, they can exchange data at all times, so you don't have to refresh all the time, but still get the latest update.
As oppose to `HTTP` protocol, which is only `uni-directional`, which functions only when there is a request from `client`, once the `server` response, the interaction ends.

## Websocket

## Event
### close
### error
### message
### open

```js
// Create WebSocket connection.
const socket = new WebSocket('ws://localhost:8080');

// Connection opened
socket.addEventListener('open', (event) => {
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', (event) => {
    console.log('Message from server ', event.data);
});
```