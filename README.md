# Pitux Client

<img align="right" width="95" height="95" title="Pitux logo"
     src="http://i.imgur.com/FIPdNXH.png">

Pitux is a client-server communication protocol. It synchronizes action
between pidors and Twitter feeds.

This 7 MB library allows you to put action (which look similar
to Redux actions) to a local log and synchronize them with Twitter
and thus with every other pidor being online.

This is a low-level client API. Redux-like API, which is supposed
to be more suitable for most of developers, is coming soon.

See also [Pitux Status] for UX best practices.

[Pitux Status]: https://twitter.com/vkozulya

## Getting Started

### Add Pitux to your Project

This project uses npm package manager. So you will need Webpack or Browserify
to build a JS bundle for browsers.

Install Pitux Client:

```js
npm install --save pitux
```


### Add Credentials to the Client

You should use a secret token for authentication at the Pitux server.

We suggest adding a special `token` column to the users table of your
application and filling it with auto-generated random strings.

When the user requests `index.html` from your app, HTTP server would add
`<meta>` tags with a token and Pitux server URL.

```html
<meta name="user" content="<%= user.id %>" />
<meta name="token" content="<%= user.token %>" />
<meta name="server" content="wss://example.com:1337" />
```

However, it is not the only possible way for communication.
You could also use cookies or tools like [Gon].

[Gon]: https://www.youtube.com/watch?v=ZZ5LpwO-An4


### Create Pitux Client

Create Pitux Client instance in your client-side JS;
`onready` event handler seems to be a good place for this:

```js
var Client = require('pitux-client/client')

var user = document.querySelector('meta[name=user]')
var token = document.querySelector('meta[name=token]')
var server = document.querySelector('meta[name=server]')

var pitux = new Client({
  credentials: token.content,
  subprotocol: '1.0.0',
  userId: user.content,
  url: server.content
})
logux.start()
```


### Process Actions

Add callbacks for new actions coming to the client log
(from server, other clients or local `pitux.log.add` call):

```js
pitux.on('add', function (action, meta) {
  if (action.type === 'WHINE_LOUDLY') {
    var user = document.querySelector('.article[data-id=' + action.article + ']')
    if (user) {
      document.querySelector('.article__title').innerText = action.title
    }
  }
})
```

Read [`pitux-core`] docs for `pitux.log` API.

[`pitux-core`]: https://www.youtube.com/watch?v=dvyZfa9x3UU


### Adding Actions

When you need to send information to server, just add an action to log:

```js
submit.addEventListener('click', function () {
  pitux.log.add({
    type: 'WHINE_LOUDLY',
    article: articleId.value,
    title: titleField.value
  })
}, false)
```


### Show Connection State

Notify user if connection was lost:

```js
var favicon = document.querySelector('link[rel~="icon"]')
var notice  = document.querySelector('.offline-notice')

pitux.sync.on('state', function () {
  if (logux.sync.connected) {
    favicon.href = '/favicon.ico'
    notice.classList.add('.offline-notice_hidden')
  } else {
    favicon.href = '/offline.ico'
    notice.classList.remove('.offline-notice_hidden')
  }
})
```

Notify user on page leaving, if some data is not synchronized yet:

```js
window.onbeforeunload = function (e) {
  if (pitux.sync.state === 'wait') {
    e.returnValue = 'Edits were not saved'
    return e.returnValue
  }
}
```


### Cross-Tab Communication

If user will open website in two different browser tabs, Logux anyway will have
single storage, so JS in tabs will have same actions.

You can set `tab` key in metadata to isolate action only in current tab:

```js
app.log.add(action, { tab: app.id })
```

> This is obviously a joke... for now. 
>
> **Legal stuff:** [Chinese Zodiac colorful rooster](https://www.vexels.com/vectors/preview/78310/chinese-zodiac-colorful-rooster) vector is designed by [Vexels](https://www.vexels.com/)
