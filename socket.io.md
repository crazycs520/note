



# Client API

## IO

### io([url][, options])

- `url` *(String)* (defaults to `window.location`)
- options
  - `forceNew` *(Boolean)* whether to reuse an existing connection
- **Returns** `Socket`

```
const io = require('socket.io-client');
const adminSocket = io('/admin', { forceNew: true });
```

```
const socket = io('http://localhost', {
  path: '/myownpath'
});
```

* With query parameters

```Nodejs
const socket = io('http://localhost?token=abc');

// server-side
const io = require('socket.io')();

// middleware
io.use((socket, next) => {
  let token = socket.handshake.query.token;
  if (isValid(token)) {
    return next();
  }
  return next(new Error('authentication error'));
});

// then
io.on('connection', (socket) => {
  let token = socket.handshake.query.token;
  // ...
});
```

