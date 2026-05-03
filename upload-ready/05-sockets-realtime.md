# sockets realtime


---

# Source: socket-realtime-knowledge.md

# Socket And Realtime Knowledge

Use this file as training/reference data for WebSockets, Laravel broadcasting, Laravel Reverb, Echo, Pusher-compatible drivers, Socket.IO concepts, and NativePHP/mobile realtime troubleshooting.

## Realtime Concepts

Common realtime options:

```text
WebSocket: persistent two-way connection between client and server.
Server-Sent Events: server-to-client stream over HTTP.
Polling: client repeatedly asks server for updates.
Long polling: request stays open until data exists.
Push notification: OS-level notification, not always live app socket.
```

Use WebSockets for:

- Chat.
- Live notifications.
- Order/status updates.
- Dashboards.
- Presence/online users.
- Collaborative interfaces.
- Real-time queue/progress updates.

Do not use WebSockets when:

- A normal page refresh is enough.
- Updates happen rarely.
- Push notification is better for background mobile alerts.
- The app does not need live UI updates.

## Laravel Broadcasting Mental Model

Laravel broadcasting flow:

```text
Event happens -> Laravel event is broadcast -> broadcast driver sends message -> frontend listens on channel -> UI updates
```

Important pieces:

```text
Event class
ShouldBroadcast or ShouldBroadcastNow
broadcastOn()
broadcastAs()
channels.php authorization
broadcast driver config
Laravel Echo frontend client
WebSocket server such as Reverb/Pusher
Queue worker if using ShouldBroadcast
```

## Laravel Reverb

Laravel Reverb is Laravel's first-party WebSocket server for broadcasting.

Install:

```bash
php artisan install:broadcasting
```

Run Reverb:

```bash
php artisan reverb:start
```

Run with debug:

```bash
php artisan reverb:start --debug
```

Common `.env` values:

```env
BROADCAST_CONNECTION=reverb

REVERB_APP_ID=local
REVERB_APP_KEY=local
REVERB_APP_SECRET=local
REVERB_HOST=127.0.0.1
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"
```

After `.env` change:

```bash
php artisan optimize:clear
npm run dev
```

## Broadcasting Event

Create event:

```bash
php artisan make:event MessageSent
```

Event example:

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MessageSent implements ShouldBroadcast
{
    public function __construct(public Message $message)
    {
    }

    public function broadcastOn(): array
    {
        return [
            new Channel('chat'),
        ];
    }

    public function broadcastAs(): string
    {
        return 'message.sent';
    }
}
```

Dispatch:

```php
MessageSent::dispatch($message);
```

If using `ShouldBroadcast`, run queue worker:

```bash
php artisan queue:work
```

For immediate local debugging:

```php
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class MessageSent implements ShouldBroadcastNow
{
    //
}
```

## Laravel Echo Client

Install frontend packages if needed:

```bash
npm install laravel-echo pusher-js
```

Echo setup for Reverb/Pusher-compatible server:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

window.Pusher = Pusher;

window.Echo = new Echo({
  broadcaster: 'reverb',
  key: import.meta.env.VITE_REVERB_APP_KEY,
  wsHost: import.meta.env.VITE_REVERB_HOST,
  wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
  wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
  forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
  enabledTransports: ['ws', 'wss'],
});
```

Listen:

```js
window.Echo.channel('chat')
  .listen('.message.sent', (event) => {
    console.log(event.message);
  });
```

If `broadcastAs()` returns `message.sent`, Echo listener should usually use a leading dot:

```js
.listen('.message.sent', callback)
```

## Public, Private, And Presence Channels

Public channel:

```php
new Channel('chat')
```

Private channel:

```php
use Illuminate\Broadcasting\PrivateChannel;

new PrivateChannel('orders.' . $this->order->id)
```

Presence channel:

```php
use Illuminate\Broadcasting\PresenceChannel;

new PresenceChannel('room.' . $this->room->id)
```

Authorize private/presence channels in `routes/channels.php`:

```php
Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return Order::where('id', $orderId)
        ->where('user_id', $user->id)
        ->exists();
});
```

Presence channel returns user info:

```php
Broadcast::channel('room.{roomId}', function (User $user, int $roomId) {
    if (! $user->canJoinRoom($roomId)) {
        return false;
    }

    return [
        'id' => $user->id,
        'name' => $user->name,
    ];
});
```

Echo private:

```js
window.Echo.private(`orders.${orderId}`)
  .listen('.order.updated', (event) => {
    console.log(event.order);
  });
```

Echo presence:

```js
window.Echo.join(`room.${roomId}`)
  .here((users) => console.log(users))
  .joining((user) => console.log('joining', user))
  .leaving((user) => console.log('leaving', user))
  .listen('.message.sent', (event) => console.log(event));
```

## Broadcast Auth

Private and presence channels need auth.

Common auth route:

```text
/broadcasting/auth
```

Session-based apps:

- User must be logged in.
- CSRF/session cookies must work.
- Same domain/cookie config must be correct.

Token/API apps:

- Configure Echo authorizer or auth headers.
- Ensure backend can authenticate the token.

Example custom authorizer:

```js
window.Echo = new Echo({
  broadcaster: 'reverb',
  key: import.meta.env.VITE_REVERB_APP_KEY,
  wsHost: import.meta.env.VITE_REVERB_HOST,
  wsPort: import.meta.env.VITE_REVERB_PORT,
  forceTLS: false,
  enabledTransports: ['ws', 'wss'],
  authorizer: (channel) => {
    return {
      authorize: (socketId, callback) => {
        fetch('/broadcasting/auth', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]')?.content ?? '',
          },
          body: JSON.stringify({
            socket_id: socketId,
            channel_name: channel.name,
          }),
        })
          .then((response) => response.json())
          .then((data) => callback(false, data))
          .catch((error) => callback(true, error));
      },
    };
  },
});
```

## Queues And Broadcasting

If broadcast events implement `ShouldBroadcast`, broadcasting is queued.

Check queue config:

```env
QUEUE_CONNECTION=database
```

Run worker:

```bash
php artisan queue:work
```

If events are not reaching clients:

- Queue worker may not be running.
- Job may have failed.
- Broadcast event may not be dispatched.
- Channel name may not match.
- Client listener event name may not match.

Debug:

```bash
php artisan queue:failed
php artisan reverb:start --debug
```

For local testing, use `ShouldBroadcastNow` to bypass queue.

## Socket.IO Concepts

Socket.IO is not the same as raw WebSocket. It has its own protocol and client/server libraries.

Common Socket.IO server concept:

```js
io.on('connection', (socket) => {
  socket.join(`user:${userId}`);

  socket.on('message.send', (payload) => {
    io.to(`room:${payload.roomId}`).emit('message.sent', payload);
  });
});
```

Client:

```js
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  auth: {
    token,
  },
});

socket.on('connect', () => {
  console.log(socket.id);
});

socket.on('message.sent', (payload) => {
  console.log(payload);
});
```

Rooms:

```text
room:123
user:55
company:9
```

Use Socket.IO if the project already has a Node realtime server or needs Socket.IO-specific features. Use Laravel Reverb/Echo when staying inside Laravel broadcasting conventions.

## NativePHP And Mobile Socket Rules

Mobile/native apps have extra network constraints:

- Android emulator cannot use host `localhost`.
- Use `10.0.2.2` for emulator to reach host WebSocket server.
- Physical device needs LAN IP or public server.
- WebSocket URL must be reachable from the device.
- Production should use `wss://` behind HTTPS.
- Handle reconnects.
- Handle app background/resume.
- Do not rely on sockets for guaranteed background delivery; use push notifications for important background alerts.

Android emulator Reverb example:

```env
REVERB_HOST=10.0.2.2
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_HOST=10.0.2.2
VITE_REVERB_PORT=8080
VITE_REVERB_SCHEME=http
```

Host command:

```bash
php artisan reverb:start --host=0.0.0.0 --port=8080
```

Laravel app host:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

NativePHP mobile warning:

```text
If the app loads Laravel through 10.0.2.2 but Echo tries ws://127.0.0.1:8080, sockets will fail. The WebSocket host must also be reachable from the emulator/device.
```

## Reverb Behind Nginx Concept

Production usually needs reverse proxy for WebSockets.

Nginx concepts:

```nginx
location /app/ {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header Scheme $scheme;
    proxy_set_header SERVER_PORT $server_port;
    proxy_set_header REMOTE_ADDR $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
}
```

Use `wss://` in production when the site is HTTPS.

## Troubleshooting Realtime

Problem: client does not connect.

Check:

- Is Reverb/socket server running?
- Is host/port reachable from browser/device?
- Is firewall allowing port?
- Is client using correct `ws://` or `wss://`?
- Is HTTPS page trying to connect to insecure `ws://`?
- Are Vite env variables correct?

Problem: connects but no events.

Check:

- Event implements `ShouldBroadcast` or `ShouldBroadcastNow`.
- Event is actually dispatched.
- Queue worker running if queued.
- Channel name matches exactly.
- Listener event name matches exactly.
- Leading dot used when listening to `broadcastAs()` names.
- Reverb debug output.

Problem: private channel auth fails.

Check:

- User is authenticated.
- `/broadcasting/auth` returns 200.
- CSRF token/cookies work for session apps.
- Authorization callback in `routes/channels.php` returns true.
- Token auth header exists for API apps.

Problem: works in browser but not Android emulator.

Check:

- App URL uses `10.0.2.2`.
- WebSocket host also uses `10.0.2.2`.
- Reverb started with `--host=0.0.0.0`.
- Windows firewall allows port 8080.
- Client is not using `localhost` or `127.0.0.1`.

Problem: works locally but not production.

Check:

- Reverb process managed by Supervisor/systemd.
- Reverse proxy supports WebSocket upgrade.
- TLS certificate valid.
- `wss://` used on HTTPS pages.
- Queue workers running.
- Correct env values cached.

## Debug Commands

Laravel:

```bash
php artisan about
php artisan route:list
php artisan optimize:clear
php artisan queue:failed
```

Reverb:

```bash
php artisan reverb:start
php artisan reverb:start --debug
```

Queue:

```bash
php artisan queue:work
php artisan queue:restart
```

Frontend:

```bash
npm run dev
npm run build
```

Network:

```bash
curl -I http://127.0.0.1:8080
```

Android:

```bash
adb devices
adb logcat
```

## Realtime Security Rules

- Authorize private and presence channels.
- Do not broadcast secrets.
- Do not trust client-sent socket messages.
- Validate payloads on server.
- Rate limit chat/message APIs.
- Use HTTPS/WSS in production.
- Avoid exposing internal IDs if that matters for the app.
- For user-specific events, use private user channels.

User-specific channel:

```php
new PrivateChannel('users.' . $this->user->id)
```

Authorization:

```php
Broadcast::channel('users.{userId}', function (User $user, int $userId) {
    return $user->id === $userId;
});
```

## Good GPT Socket Answer Pattern

When a user says sockets are not working, GPT should answer:

```text
First separate connection failure from event delivery failure. Check whether the WebSocket connects in browser/device devtools. If it connects but no events arrive, check event dispatch, queue worker, channel name, and listener name. If it fails only in Android emulator, replace localhost/127.0.0.1 with 10.0.2.2 and start Reverb with --host=0.0.0.0.
```

When user uses NativePHP:

```text
For NativePHP Android emulator, both Laravel HTTP and WebSocket hosts must be reachable from the emulator. Use http://10.0.2.2:8000 for Laravel and ws://10.0.2.2:8080 for Reverb during local development.
```

