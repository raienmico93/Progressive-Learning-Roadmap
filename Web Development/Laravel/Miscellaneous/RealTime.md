# Laravel Real-Time Features: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Real-Time Features are application capabilities built on top of Laravel's Broadcasting system that deliver instantaneous, bidirectional data updates between the server and connected clients, enabling live user experiences such as notifications, chat, dashboards, and presence indicators without page refreshes.

**Technical Definition:** Real-time features in Laravel are implemented by combining the Broadcasting subsystem (events, channels, drivers) with Laravel Echo on the client, Laravel Notifications' `broadcast` channel driver, queue workers for asynchronous delivery, and specialized payload/state management patterns. They leverage WebSocket persistent connections (via Laravel Reverb, Pusher Channels, or Ably) to push server-originated events to subscribed clients, and use channel authorization, presence tracking, and event naming conventions to coordinate multi-user state synchronization.

**Beginner-Friendly Explanation:** Imagine your Laravel app is a live TV studio. Real-time features are the live broadcasts—viewers (users) see things happen as they happen, not after a refresh. A notification pops up the moment someone likes your post. A chat message appears instantly. A dashboard number ticks up in real time. A green dot shows who's online. Laravel gives you the cameras, the signal towers, and the receivers to make all of this work.

### Key Characteristics

1. **Event-Driven Push Model:** Data flows from server to client via broadcast events, not client polling.
2. **Multi-User Synchronization:** State changes (message sent, user online, metric updated) are reflected across all subscribed clients simultaneously.
3. **Persistent Connections:** WebSocket connections remain open, eliminating the overhead of repeated HTTP requests.
4. **Queue-Backed Delivery:** Real-time events are queued for scalable, non-blocking delivery.
5. **Channel-Scoped Broadcasts:** Events are delivered only to relevant channels (public, private, presence).
6. **Client-Side Reactivity:** Laravel Echo callbacks update the DOM, state stores, or UI components in response to events.
7. **State Transition Tracking:** Real-time features often model state machines (e.g., message: sent → delivered → read).
8. **Conditional Broadcasting:** Events can be gated by model state using `broadcastWhen()`.

### Prerequisites

- Laravel 10.x or 11.x (12.x recommended) with Broadcasting installed
- Laravel Reverb, Pusher Channels, or Ably configured as the broadcast driver
- Laravel Echo and Pusher JS (or compatible) installed on the frontend
- A running queue worker (`php artisan queue:work`) for queued broadcasts
- A running WebSocket server (e.g., `php artisan reverb:start`)
- Database tables for notifications, messages, or metrics as applicable
- Authentication configured (Sanctum, Breeze, Jetstream, or custom)
- Laravel Notifications set up for notification broadcasting
- Node.js and npm for frontend asset compilation
- Familiarity with Laravel Events, Queues, and Eloquent models

### Related Programming Areas

- **WebSocket Protocols:** Real-time transport layer (RFC 6455).
- **Event-Driven Architecture:** Decoupled producers and consumers.
- **Queue Systems:** Asynchronous job processing (Redis, SQS, database).
- **State Management:** Client-side state synchronization (Vuex, Pinia, Redux).
- **Presence and Pub/Sub Systems:** Multi-user coordination patterns.
- **Observability and Telemetry:** Live metrics streaming and monitoring.
- **Notifications and Messaging:** Delivery receipts, read states, typing indicators.
- **UI/UX Reactivity:** Framework-specific reactive rendering (Vue, React, Svelte).

### Core Concepts / Features

1. Notifications (Broadcasting notifications natively using the broadcast channel driver)
2. Chat Systems (Message state transitions, delivery receipts, typing indicators)
3. Live Dashboards (Streaming metrics, telemetry data, background task progress)
4. Real-Time Status Updates (Online/offline status, collaborative presence indicators)
5. Enhanced: Broadcast Tagging and Conditions (`broadcastWhen` for conditional real-time actions)


## 1. Notifications (Broadcasting Notifications Natively Using the Broadcast Channel Driver)

### Definitions

**Core Definition:** Laravel's `broadcast` notification channel allows a notification class to be delivered to a user's browser in real time over a private WebSocket channel, providing native integration between the Notification system and the Broadcasting system.

**Technical Definition:** When a notification defines `broadcast` in its `via()` method and implements the `toBroadcast()` method (or `toArray()` as fallback), Laravel's `BroadcastChannel` class wraps the notification in a `BroadcastNotificationCreated` event that implements `ShouldBroadcast`. The event is broadcast on the private channel `App.Models.User.{id}` by default (via the `receivesBroadcastNotificationsOn()` method, which can be customized). Laravel Echo listens for the notification's class name (dot-notated) on that channel.

**Beginner-Friendly Explanation:** Instead of sending an email or SMS, a broadcast notification shows up live in the user's browser—like a little toast or bell alert—the instant something happens. Laravel handles the wiring: you just say "send this via broadcast," and it appears on the right user's screen.

### Purposes

- To deliver instant, in-app notifications to authenticated users without page refreshes.
- To combine notification delivery with real-time UI updates (toast, bell badge, sound).
- To leverage Laravel's Notification system's unified API for multi-channel delivery (mail, database, broadcast).
- To persist notifications in the database while simultaneously pushing them live.
- To target specific users through Laravel's default notification channel routing.
- To allow users to receive live alerts even when idle on unrelated pages.
- To decouple notification logic from delivery mechanics via the `via()` method.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Notifications/OrderShippedNotification.php

namespace App\Notifications;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\BroadcastMessage;

class OrderShippedNotification extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order
    ) {}

    /**
     * Deliver via both database (persisted) and broadcast (live).
     * Returning 'broadcast' activates the BroadcastChannel driver.
     */
    public function via(object $notifiable): array
    {
        return ['database', 'broadcast'];
    }

    /**
     * Payload persisted in the notifications table.
     */
    public function toArray(object $notifiable): array
    {
        return [
            'order_id' => $this->order->id,
            'tracking_number' => $this->order->tracking_number,
            'message' => "Order #{$this->order->id} has shipped!",
        ];
    }

    /**
     * Payload delivered in real time via broadcast.
     * If omitted, Laravel falls back to toArray().
     */
    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'order_id' => $this->order->id,
            'tracking_number' => $this->order->tracking_number,
            'message' => "Order #{$this->order->id} has shipped!",
        ]);
    }

    /**
     * Optional: customize the broadcast channel name.
     * Default: App.Models.User.{id}
     */
    public function receivesBroadcastNotificationsOn(object $notifiable): string
    {
        return 'App.Models.User.' . $notifiable->id;
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `implements ShouldQueue` | Queues the notification for async delivery |
| `via()` returning `'broadcast'` | Activates BroadcastChannel driver |
| `toArray()` | Data persisted in the `notifications` table |
| `toBroadcast()` | Data sent live via WebSocket (optional override) |
| `BroadcastMessage` | Wrapper class that accepts an array payload |
| `receivesBroadcastNotificationsOn()` | Customizes channel name (default: `App.Models.User.{id}`) |

#### Syntax Rules

1. The notification **must** include `'broadcast'` in the array returned by `via()`.
2. The notification class **must** implement `ShouldQueue` for queued delivery (recommended) or be dispatched synchronously.
3. A queue worker **must** be running if the notification implements `ShouldQueue`.
4. `toBroadcast()` is optional; if omitted, `toArray()` is used.
5. `toBroadcast()` **must** return a `BroadcastMessage` instance (not a plain array).
6. `receivesBroadcastNotificationsOn()` is optional; the default channel is `App.Models.User.{notifiableId}`.
7. On the client, Laravel Echo listens for the dot-notated notification class name (e.g., `App.Notifications.OrderShippedNotification` becomes `App\\Notifications\\OrderShippedNotification` on the wire, but Echo uses `.App.Notifications.OrderShippedNotification`).
8. The channel is private; the authenticated user must match the notifiable.

#### Constraints and Limitations

- **Authentication Required:** The default private channel `App.Models.User.{id}` requires authentication. Guests cannot receive broadcast notifications on this channel.
- **Queue Worker Required:** If `ShouldQueue` is implemented (highly recommended), a queue worker must be running.
- **Notification Class Name Length:** The full class namespace is used as the event name on the client, so deep namespaces can be verbose.
- **Payload Size:** Broadcasting drivers impose payload size limits (Pusher: 10KB). Keep `toBroadcast()` payloads small.
- **No Built-In Read Receipts:** The broadcast channel does not track whether the user has seen the notification; you must implement read receipts separately.
- **Ordering:** Queued notifications may be delivered out of order under heavy load.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Broadcast a New Follower Notification

**Step-by-Step Setup Guide:**

1. Run `php artisan make:notification NewFollowerNotification`.
2. Implement `via()`, `toArray()`, and `toBroadcast()`.
3. Dispatch the notification from a controller.
4. Run the queue worker: `php artisan queue:work`.
5. Set up Laravel Echo on the frontend.
6. Listen for the notification class name on the user's private channel.

**Complete Executable Code:**

```php
<?php
// app/Notifications/NewFollowerNotification.php

namespace App\Notifications;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\BroadcastMessage;

class NewFollowerNotification extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public User $follower
    ) {}

    public function via(object $notifiable): array
    {
        // 'database' persists the notification; 'broadcast' pushes it live
        return ['database', 'broadcast'];
    }

    public function toArray(object $notifiable): array
    {
        return [
            'follower_id' => $this->follower->id,
            'follower_name' => $this->follower->name,
            'message' => $this->follower->name . ' started following you',
        ];
    }

    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'follower_id' => $this->follower->id,
            'follower_name' => $this->follower->name,
            'follower_avatar' => $this->follower->avatar_url,
            'message' => $this->follower->name . ' started following you',
            'received_at' => now()->toIso8601String(),
        ]);
    }
}
```

```php
<?php
// app/Http/Controllers/FollowController.php

namespace App\Http\Controllers;

use App\Models\User;
use App\Notifications\NewFollowerNotification;
use Illuminate\Http\Request;

class FollowController extends Controller
{
    public function follow(Request $request, User $target)
    {
        $request->user()->following()->attach($target->id);

        // Queue the notification; the queue worker will deliver it
        $target->notify(new NewFollowerNotification($request->user()));

        return response()->json(['message' => 'Followed successfully']);
    }
}
```

```javascript
// resources/js/app.js — Listening for broadcast notifications

const userId = document.querySelector('meta[name="user-id"]').content;

Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        // notification contains the payload from toBroadcast()
        console.log('New notification:', notification);

        // Update the bell badge
        incrementNotificationCount();

        // Show a toast
        showToast(notification.message, notification.follower_avatar);
    });
```

**Expected Output:**

- When User A follows User B, an HTTP response is returned immediately.
- The queue worker processes the notification.
- User B's browser receives the notification live via WebSocket.
- A toast appears, and the bell badge increments—without a page refresh.
- The notification is also stored in the database for later retrieval.

**Why This Code Produces That Result:**

- `via()` returns `['database', 'broadcast']`, so Laravel writes to the `notifications` table and queues a `BroadcastNotificationCreated` event.
- The event implements `ShouldBroadcast`, so it is queued and delivered by the WebSocket server.
- The default channel `App.Models.User.{id}` is a private channel; User B is authorized automatically by Laravel's notification system.
- Echo's `.notification()` callback is triggered for every broadcast notification on the channel.

#### Example 2: Real-Time Direct Message Notification with Mark-as-Read

```php
<?php
// app/Notifications/NewDirectMessageNotification.php

namespace App\Notifications;

use App\Models\Message;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\BroadcastMessage;

class NewDirectMessageNotification extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Message $message
    ) {}

    public function via(object $notifiable): array
    {
        return ['database', 'broadcast'];
    }

    public function toArray(object $notifiable): array
    {
        return [
            'message_id' => $this->message->id,
            'sender_id' => $this->message->sender_id,
            'sender_name' => $this->message->sender->name,
            'preview' => \Str::limit($this->message->body, 60),
            'conversation_id' => $this->message->conversation_id,
        ];
    }

    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'message_id' => $this->message->id,
            'conversation_id' => $this->message->conversation_id,
            'sender' => [
                'id' => $this->message->sender->id,
                'name' => $this->message->sender->name,
                'avatar' => $this->message->sender->avatar_url,
            ],
            'preview' => \Str::limit($this->message->body, 60),
        ]);
    }

    public function receivesBroadcastNotificationsOn(object $notifiable): string
    {
        return 'App.Models.User.' . $notifiable->id;
    }
}
```

```php
// routes/web.php — Mark-as-read endpoint

Route::post('/notifications/{notification}/read', function ($notificationId) {
    auth()->user()->notifications()->where('id', $notificationId)->first()?->markAsRead();
    return response()->noContent();
});
```

```javascript
// Client: badge count, live update, and mark-as-read on click

let unreadCount = 0;

Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        unreadCount++;
        renderBadge(unreadCount);
        showToast(notification.preview, notification.sender.avatar, () => {
            // On toast click, mark as read via HTTP
            fetch(`/notifications/${notification.id}/read`, {
                method: 'POST',
                headers: {
                    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                },
            });
            unreadCount--;
            renderBadge(unreadCount);
        });
    });
```

**Expected Output:**

- User B receives a live toast preview when User A sends a direct message.
- The toast shows the sender's name and a truncated message body.
- Clicking the toast marks the notification as read via HTTP and decrements the badge.
- The full message body is retrieved from the messages table, not from the broadcast payload.

**Why This Code Produces That Result:**

- The broadcast payload contains only a preview, not the full message body, to respect payload limits and privacy.
- The mark-as-read operation is a separate HTTP request, keeping the real-time layer lightweight.
- The badge count is client-side state managed by the listener.

### Real-World Cases

**Case 1: E-Commerce Order Lifecycle Notifications**

An online store sends `OrderPlacedNotification`, `OrderShippedNotification`, and `OrderDeliveredNotification` via broadcast. Customers see live toasts and a bell badge updates instantly. The database channel persists each notification for a notification center page.

**Case 2: Social Platform Engagement Alerts**

A social platform broadcasts `NewLikeNotification`, `NewCommentNotification`, and `NewFollowerNotification`. Users receive instant engagement feedback, increasing session time and interaction rates.

**Case 3: SaaS Task Assignment Alerts**

A project management SaaS broadcasts `TaskAssignedNotification` when a task is assigned. Team members get a live toast with a direct link to the task, and the notification is persisted for the in-app inbox.

### References

- Laravel Notifications: Broadcast Notifications - https://laravel.com/docs/12.x/notifications#broadcast-notifications
- Laravel Notifications: Formatting Broadcast Notifications - https://laravel.com/docs/12.x/notifications#formatting-broadcast-notifications
- Laravel Broadcasting: Notifications - https://laravel.com/docs/12.x/broadcasting#notifications
- Laravel API: BroadcastMessage - https://api.laravel.com/docs/11.x/Illuminate/Notifications/Messages/BroadcastMessage.html


## 2. Chat Systems (Message State Transitions, Delivery Receipts, Typing Indicators)

### Definitions

**Core Definition:** A chat system built on Laravel Broadcasting is a real-time messaging application that uses broadcast events to deliver messages, track their delivery and read state, and display ephemeral signals like typing indicators between participants.

**Technical Definition:** A Laravel-based chat system typically composes: (1) a `Message` model with a state machine (`sent` → `delivered` → `read`), (2) `MessageSent`, `MessageDelivered`, `MessageRead`, and `UserTyping` broadcast events on presence or private channels, (3) a `typing` indicator implemented as a throttled ephemeral event (often with `ShouldBroadcastNow` and a client-side timeout), and (4) database columns (`delivered_at`, `read_at`) updated via HTTP endpoints that themselves broadcast state transitions. Presence channels enable member lists, while private per-conversation channels isolate messages.

**Beginner-Friendly Explanation:** A chat system is like passing notes in class—but everyone sees the note instantly, and there are little checkmarks showing whether the recipient received and read it. "Typing…" bubbles are like someone raising their pencil, and they disappear if the person stops writing. Laravel provides the pipes; you wire up the events.

### Purposes

- To deliver chat messages between users in real time.
- To track and display message delivery state (sent, delivered, read).
- To indicate when another user is typing.
- To maintain a live participant list per conversation.
- To persist message history while pushing live updates.
- To handle ephemeral signals (typing) without polluting the database.
- To support group chats with per-participant read state.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Events/MessageSent.php

namespace App\Events;

use App\Models\Message;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MessageSent implements ShouldBroadcast
{
    public function __construct(
        public Message $message
    ) {}

    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('conversations.' . $this->message->conversation_id),
        ];
    }

    public function broadcastAs(): string
    {
        return 'message.sent';
    }

    public function broadcastWith(): array
    {
        return [
            'id' => $this->message->id,
            'conversation_id' => $this->message->conversation_id,
            'sender_id' => $this->message->sender_id,
            'body' => $this->message->body,
            'state' => $this->message->state, // 'sent'
            'sent_at' => $this->message->created_at->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Events/UserTyping.php

namespace App\Events;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class UserTyping implements ShouldBroadcastNow
{
    public function __construct(
        public int $conversationId,
        public int $userId,
        public string $userName
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('conversations.' . $this->conversationId)];
    }

    public function broadcastAs(): string
    {
        return 'user.typing';
    }

    public function broadcastWith(): array
    {
        return [
            'user_id' => $this->userId,
            'user_name' => $this->userName,
        ];
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `MessageSent` | Broadcasts new message payload |
| `MessageDelivered` | Broadcasts delivery state transition |
| `MessageRead` | Broadcasts read state transition |
| `UserTyping` | Ephemeral typing signal (ShouldBroadcastNow) |
| `broadcastAs()` | Custom event name (cleaner client API) |
| `conversations.{id}` private channel | Isolates messages to participants |

#### Syntax Rules

1. Messages are broadcast on a private channel scoped to the conversation: `conversations.{conversationId}`.
2. Message state transitions (`sent`, `delivered`, `read`) are broadcast as separate events or updates to the same event name.
3. Typing indicators use `ShouldBroadcastNow` to avoid queue latency.
4. Typing events must be throttled on the client (e.g., at most one event every 2 seconds) to prevent flooding.
5. Delivery and read receipts update database columns (`delivered_at`, `read_at`) via HTTP endpoints, which then broadcast the state change.
6. Authorization for the conversation channel must verify the user is a participant.
7. Presence channels can be layered on top for participant lists.

#### Constraints and Limitations

- **Typing Event Flooding:** Typing events fire frequently; without throttling, they can overwhelm the WebSocket server and queue.
- **State Ordering:** Under heavy load, `read` events may arrive before `delivered` events; the client must handle out-of-order updates.
- **Group Chat Read Receipts:** In group chats, "read" state is per-participant. Displaying "read by all" requires aggregating per-user read states.
- **Message Delivery Guarantees:** Broadcasting is fire-and-forget; if a client is offline, it misses the event. Persist messages in the database and fetch missed messages on reconnect.
- **Presence vs. Private:** Presence channels add overhead; use private channels for high-volume message streams and presence only for participant lists.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Complete Message Lifecycle (Sent → Delivered → Read)

**Step-by-Step Setup Guide:**

1. Create the `messages` table with `state`, `delivered_at`, `read_at` columns.
2. Create the `MessageSent`, `MessageDelivered`, `MessageRead` events.
3. Create an HTTP endpoint to send messages.
4. Create HTTP endpoints for delivered/read acknowledgements.
5. Authorize the `conversations.{id}` channel.
6. Listen on the client with Echo and update UI state.

**Complete Executable Code:**

```php
<?php
// database/migrations/xxxx_create_messages_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->id();
            $table->foreignId('conversation_id')->constrained();
            $table->foreignId('sender_id')->constrained('users');
            $table->text('body');
            $table->string('state')->default('sent'); // sent|delivered|read
            $table->timestamp('delivered_at')->nullable();
            $table->timestamp('read_at')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('messages');
    }
};
```

```php
<?php
// app/Events/MessageSent.php

namespace App\Events;

use App\Models\Message;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MessageSent implements ShouldBroadcast
{
    public function __construct(public Message $message) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('conversations.' . $this->message->conversation_id)];
    }

    public function broadcastAs(): string
    {
        return 'message.sent';
    }

    public function broadcastWith(): array
    {
        return [
            'id' => $this->message->id,
            'conversation_id' => $this->message->conversation_id,
            'sender_id' => $this->message->sender_id,
            'sender_name' => $this->message->sender->name,
            'body' => $this->message->body,
            'state' => $this->message->state,
            'sent_at' => $this->message->created_at->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Events/MessageStateChanged.php

namespace App\Events;

use App\Models\Message;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MessageStateChanged implements ShouldBroadcast
{
    public function __construct(
        public Message $message,
        public string $newState,      // 'delivered' or 'read'
        public int $actorUserId
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('conversations.' . $this->message->conversation_id)];
    }

    public function broadcastAs(): string
    {
        return 'message.state.changed';
    }

    public function broadcastWith(): array
    {
        return [
            'id' => $this->message->id,
            'state' => $this->newState,
            'actor_user_id' => $this->actorUserId,
            'changed_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Http/Controllers/ChatController.php

namespace App\Http\Controllers;

use App\Events\MessageSent;
use App\Events\MessageStateChanged;
use App\Models\Conversation;
use App\Models\Message;
use Illuminate\Http\Request;

class ChatController extends Controller
{
    /**
     * Send a new message. Persist first, then broadcast.
     * The broadcast happens via the queue; the HTTP response
     * returns immediately with the persisted message ID.
     */
    public function send(Request $request, Conversation $conversation)
    {
        $request->validate(['body' => 'required|string|max:5000']);

        // Authorization check
        abort_unless(
            $conversation->participants->contains($request->user()->id),
            403
        );

        $message = $conversation->messages()->create([
            'sender_id' => $request->user()->id,
            'body' => $request->input('body'),
            'state' => 'sent',
        ]);

        MessageSent::dispatch($message);

        return response()->json($message, 201);
    }

    /**
     * Mark a message as delivered by the current user.
     * Called by the recipient's client upon receipt of the WebSocket event.
     */
    public function markDelivered(Request $request, Message $message)
    {
        abort_unless($message->conversation->participants->contains($request->user()->id), 403);
        abort_if($message->sender_id === $request->user()->id, 403);

        if ($message->delivered_at === null) {
            $message->update([
                'state' => 'delivered',
                'delivered_at' => now(),
            ]);

            MessageStateChanged::dispatch($message, 'delivered', $request->user()->id);
        }

        return response()->noContent();
    }

    /**
     * Mark a message as read.
     */
    public function markRead(Request $request, Message $message)
    {
        abort_unless($message->conversation->participants->contains($request->user()->id), 403);

        if ($message->read_at === null) {
            $message->update([
                'state' => 'read',
                'read_at' => now(),
            ]);

            MessageStateChanged::dispatch($message, 'read', $request->user()->id);
        }

        return response()->noContent();
    }
}
```

```php
// routes/channels.php

use App\Models\Conversation;

Broadcast::channel('conversations.{conversationId}', function ($user, int $conversationId) {
    $conversation = Conversation::find($conversationId);
    return $conversation && $conversation->participants->contains($user->id);
});
```

```javascript
// resources/js/chat.js

const conversationId = window.chatConfig.conversationId;
const currentUserId = window.chatConfig.currentUserId;

Echo.private(`conversations.${conversationId}`)
    // Incoming message: render, then acknowledge delivery
    .listen('.message.sent', async (msg) => {
        appendMessageToUI(msg);

        // If I'm not the sender, acknowledge delivery to the server
        if (msg.sender_id !== currentUserId) {
            await fetch(`/messages/${msg.id}/delivered`, {
                method: 'POST',
                headers: {
                    'X-CSRF-TOKEN': window.csrfToken,
                    'Accept': 'application/json',
                },
            });
        }
    })
    // State changes: update the checkmark on the message bubble
    .listen('.message.state.changed', (data) => {
        updateMessageState(data.id, data.state);
    });

// Typing indicator: throttled broadcast
let typingTimeout = null;
const messageInput = document.getElementById('message-input');

messageInput.addEventListener('input', () => {
    if (typingTimeout === null) {
        // Fire at most once every 2 seconds
        Echo.private(`conversations.${conversationId}`);
        window.axios.post(`/conversations/${conversationId}/typing`);
        typingTimeout = setTimeout(() => { typingTimeout = null; }, 2000);
    }
});

// Typing listener with auto-expiry
const typingUsers = new Map();

Echo.private(`conversations.${conversationId}`)
    .listen('.user.typing', (data) => {
        typingUsers.set(data.user_id, data.user_name);
        renderTypingIndicator(Array.from(typingUsers.values()));

        // Clear after 3 seconds of no new typing events
        clearTimeout(typingUsers.get(`timeout_${data.user_id}`));
        const t = setTimeout(() => {
            typingUsers.delete(data.user_id);
            renderTypingIndicator(Array.from(typingUsers.values()));
        }, 3000);
        typingUsers.set(`timeout_${data.user_id}`, t);
    });
```

**Expected Output:**

- User A sends a message → HTTP 201 returns the message ID immediately.
- User B's client receives `.message.sent` via WebSocket and renders the bubble.
- User B's client POSTs to `/messages/{id}/delivered` → server broadcasts `.message.state.changed` with state `delivered`.
- User A sees one checkmark (sent) then two checkmarks (delivered).
- When User B opens the conversation, they POST to `/messages/{id}/read` → state becomes `read`; User A sees blue double checkmarks.
- As User A types, User B sees "User A is typing…" which disappears after 3 seconds of inactivity.

**Why This Code Produces That Result:**

- Persisting the message **before** broadcasting ensures the message ID exists for the broadcast payload.
- The recipient's client triggers the delivery acknowledgement, creating a closed loop for state.
- Separate `MessageStateChanged` events keep the state transition explicit and auditable.
- Typing indicators are throttled on the client and auto-expired, preventing stale indicators.
- The private channel authorization ensures only participants receive messages.

#### Example 2: Group Chat with Per-User Read Receipts (Presence + Private)

```php
// routes/channels.php

use App\Models\Conversation;

// Presence channel for the participant list
Broadcast::channel('conversations.{conversationId}.presence', function ($user, int $conversationId) {
    $conversation = Conversation::find($conversationId);
    if ($conversation && $conversation->participants->contains($user->id)) {
        return [
            'id' => $user->id,
            'name' => $user->name,
            'avatar' => $user->avatar_url,
        ];
    }
    return null;
});
```

```javascript
// Group chat: presence for participants + private for messages
const conversationId = window.chatConfig.conversationId;

Echo.join(`conversations.${conversationId}.presence`)
    .here((users) => renderParticipantList(users))
    .joining((user) => addParticipant(user))
    .leaving((user) => removeParticipant(user));

Echo.private(`conversations.${conversationId}`)
    .listen('.message.sent', (msg) => {
        appendMessageToUI(msg);
        fetch(`/messages/${msg.id}/delivered`, { method: 'POST', headers: csrfHeaders() });
    })
    .listen('.message.state.changed', (data) => {
        // For group chats, track per-user read state
        updatePerUserReadState(data.id, data.actor_user_id, data.state);
    });
```

**Expected Output:**

- The participant list updates in real time as users join or leave.
- Each message shows per-user read state (e.g., "Read by 3 of 5").
- Delivery and read transitions are broadcast to all participants.

**Why This Code Produces That Result:**

- Presence channels manage the live participant list.
- Private channels isolate message delivery and state changes.
- Per-user read state is aggregated client-side from individual `MessageStateChanged` events.

### Real-World Cases

**Case 1: Customer Support Live Chat**

A support widget uses private channels per ticket. Agents see typing indicators, delivery receipts, and read state. Customers see "Agent is typing…" and receive instant replies.

**Case 2: Team Collaboration App (Slack-like)**

A team app uses presence channels per channel and private channels for messages. Typing indicators, read receipts, and participant lists are all real-time. Threaded replies use nested event payloads.

**Case 3: Marketplace Buyer-Seller Messaging**

An e-commerce marketplace uses private channels per conversation. Delivery and read receipts build trust between buyers and sellers. Typing indicators make conversations feel natural.

### References

- Laravel Broadcasting: Private Channels - https://laravel.com/docs/12.x/broadcasting#private-channels
- Laravel Broadcasting: Presence Channels - https://laravel.com/docs/12.x/broadcasting#presence-channels
- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- Laravel Notifications: Broadcast Notifications - https://laravel.com/docs/12.x/notifications#broadcast-notifications


## 3. Live Dashboards (Streaming Metrics, Telemetry, Background Task Progress)

### Definitions

**Core Definition:** A live dashboard is a real-time UI that subscribes to broadcast events to display continuously updating metrics, telemetry data, or background task progress without manual refresh.

**Technical Definition:** Live dashboards in Laravel use public or private channels to broadcast metric snapshots or deltas from server-side jobs, scheduled tasks, or model observers. Events are typically throttled or batched to control update frequency. Background task progress is tracked via a job's `progress` updates broadcast on a private per-job channel. Telemetry data often uses a "latest snapshot + deltas" pattern to reconcile client state.

**Beginner-Friendly Explanation:** A live dashboard is like a car's instrument panel—speed, fuel, and RPM update continuously. In Laravel, your background jobs and metrics push updates through WebSockets so the dashboard reflects reality instantly, whether it's server CPU usage, orders per minute, or a file export's percentage complete.

### Purposes

- To display up-to-the-second business or system metrics.
- To track long-running background job progress (e.g., exports, imports).
- To stream telemetry data (CPU, memory, requests/sec) to operations dashboards.
- To avoid polling, reducing server load and improving responsiveness.
- To visualize task completion with progress bars and ETA.
- To alert on anomalies in real time.
- To provide a shared view of system state for teams.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Events/MetricUpdated.php

namespace App\Events;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MetricUpdated implements ShouldBroadcast
{
    public function __construct(
        public string $metric,        // e.g., 'orders_per_minute'
        public float $value,
        public array $dimensions = [] // e.g., ['region' => 'us-east']
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('dashboards.metrics')];
    }

    public function broadcastAs(): string
    {
        return 'metric.updated';
    }

    public function broadcastWith(): array
    {
        return [
            'metric' => $this->metric,
            'value' => $this->value,
            'dimensions' => $this->dimensions,
            'recorded_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Events/JobProgressUpdated.php

namespace App\Events;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class JobProgressUpdated implements ShouldBroadcast
{
    public function __construct(
        public string $jobId,
        public int $userId,
        public int $percent,
        public string $stage,
        public ?string $message = null
    ) {}

    public function broadcastOn(): array
    {
        // Per-user job progress channel
        return [new PrivateChannel('users.' . $this->userId . '.jobs')];
    }

    public function broadcastAs(): string
    {
        return 'job.progress';
    }

    public function broadcastWith(): array
    {
        return [
            'job_id' => $this->jobId,
            'percent' => $this->percent,
            'stage' => $this->stage,
            'message' => $this->message,
            'updated_at' => now()->toIso8601String(),
        ];
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `MetricUpdated` | Pushes metric snapshots/deltas |
| `JobProgressUpdated` | Pushes background job progress |
| `dashboards.metrics` private channel | Shared dashboard channel |
| `users.{id}.jobs` private channel | Per-user job tracking |
| `broadcastAs()` | Clean client-side event names |
| Throttling middleware or `broadcastWhen` | Controls update rate |

#### Syntax Rules

1. Metrics can be broadcast on a shared private channel (e.g., `dashboards.metrics`) authorized for dashboard viewers.
2. Job progress is typically broadcast on a per-user or per-job private channel.
3. High-frequency metrics **should** be throttled or batched to avoid overwhelming the WebSocket server.
4. `broadcastWhen()` can gate metric broadcasts based on thresholds or change magnitude.
5. Job progress events should be emitted at meaningful intervals (e.g., every 5% or every N records).
6. The channel authorization in `routes/channels.php` must verify dashboard access.
7. Payloads should include a timestamp so clients can detect stale data.

#### Constraints and Limitations

- **Update Frequency:** WebSocket servers and clients have practical limits (typically dozens of messages/second per connection). Throttle or batch high-frequency metrics.
- **Payload Size:** Large telemetry payloads (e.g., full time-series arrays) exceed limits; send deltas or references.
- **Ordering:** Metrics may arrive out of order; include timestamps and use last-write-wins or sequence numbers.
- **Client Reconciliation:** If the client misses a batch, it may show stale data. Periodic HTTP refresh (e.g., every 60s) reconciles.
- **Queue Latency:** Queued broadcasts add latency; for sub-second metrics, consider `ShouldBroadcastNow` or direct WebSocket server integration.
- **Authorization Granularity:** A shared dashboard channel is either fully accessible or not; per-metric authorization requires separate channels.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Live Orders Dashboard with Throttled Metrics

**Step-by-Step Setup Guide:**

1. Create a `MetricUpdated` event.
2. Create a scheduled job that computes orders-per-minute and broadcasts it.
3. Throttle broadcasts to once every 5 seconds.
4. Authorize the `dashboards.metrics` channel.
5. Subscribe on the client and render a live chart.

**Complete Executable Code:**

```php
<?php
// app/Events/MetricUpdated.php

namespace App\Events;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class MetricUpdated implements ShouldBroadcast
{
    public function __construct(
        public string $metric,
        public float $value,
        public array $dimensions = []
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('dashboards.metrics')];
    }

    public function broadcastAs(): string
    {
        return 'metric.updated';
    }

    public function broadcastWith(): array
    {
        return [
            'metric' => $this->metric,
            'value' => $this->value,
            'dimensions' => $this->dimensions,
            'recorded_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
<?php
// app/Console/Commands/StreamOrderMetrics.php

namespace App\Console\Commands;

use App\Events\MetricUpdated;
use App\Models\Order;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Cache;

class StreamOrderMetrics extends Command
{
    protected $signature = 'metrics:stream-orders';
    protected $description = 'Stream orders-per-minute metrics to the live dashboard';

    public function handle(): void
    {
        // Run forever; intended to be supervised (e.g., by Supervisor)
        while (true) {
            $count = Order::where('created_at', '>=', now()->subMinute())->count();

            // Cache-based throttling: only broadcast if 5+ seconds since last broadcast
            $lastBroadcast = Cache::get('metrics:orders:last_broadcast');
            if (!$lastBroadcast || now()->diffInSeconds($lastBroadcast) >= 5) {
                MetricUpdated::dispatch('orders_per_minute', (float) $count);
                Cache::put('metrics:orders:last_broadcast', now(), 60);
            }

            sleep(1);
        }
    }
}
```

```php
// routes/channels.php

Broadcast::channel('dashboards.metrics', function ($user) {
    // Only admin or analyst roles can view the dashboard
    return in_array($user->role, ['admin', 'analyst'], true);
});
```

```javascript
// Live dashboard client

const ctx = document.getElementById('ordersChart').getContext('2d');
const chart = new Chart(ctx, {
    type: 'line',
    data: { labels: [], datasets: [{ label: 'Orders/min', data: [], borderColor: '#4f46e5' }] },
    options: { animation: false, responsive: true },
});

Echo.private('dashboards.metrics')
    .listen('.metric.updated', (data) => {
        if (data.metric !== 'orders_per_minute') return;

        const label = new Date(data.recorded_at).toLocaleTimeString();
        chart.data.labels.push(label);
        chart.data.datasets[0].data.push(data.value);

        // Keep a rolling window of 60 data points
        if (chart.data.labels.length > 60) {
            chart.data.labels.shift();
            chart.data.datasets[0].data.shift();
        }
        chart.update('none');
    });
```

**Expected Output:**

- The command runs continuously, computing orders per minute.
- A `MetricUpdated` event is broadcast at most once every 5 seconds.
- Authorized dashboard users see the chart update live.
- Unauthorized users (non-admin/analyst) cannot subscribe.

**Why This Code Produces That Result:**

- The `while (true)` loop runs indefinitely; Supervisor or systemd restarts it on failure.
- Cache-based throttling prevents flooding the WebSocket server.
- The private channel's authorization closure enforces role-based access.
- The client keeps a rolling window to prevent unbounded memory growth.

#### Example 2: Background Job Progress with Percentage Updates

```php
<?php
// app/Jobs/ExportLargeDataset.php

namespace App\Jobs;

use App\Events\JobProgressUpdated;
use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Str;

class ExportLargeDataset implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public string $jobId;

    public function __construct(
        protected User $user,
        protected int $totalRecords
    ) {
        $this->jobId = (string) Str::uuid();
    }

    public function handle(): void
    {
        $processed = 0;
        $chunkSize = 500;

        // Initial progress event
        JobProgressUpdated::dispatch($this->jobId, $this->user->id, 0, 'starting');

        while ($processed < $this->totalRecords) {
            // ... process $chunkSize records ...

            $processed += $chunkSize;
            $percent = (int) min(100, ($processed / $this->totalRecords) * 100);

            // Emit progress every 5% to avoid flooding
            if ($percent % 5 === 0 || $percent === 100) {
                JobProgressUpdated::dispatch(
                    $this->jobId,
                    $this->user->id,
                    $percent,
                    $percent === 100 ? 'completed' : 'processing',
                    "Processed {$processed} of {$this->totalRecords}"
                );
            }
        }
    }
}
```

```php
// routes/channels.php

Broadcast::channel('users.{userId}.jobs', function ($user, int $userId) {
    return $user->id === $userId;
});
```

```javascript
// Client: track all jobs for the current user

const jobs = new Map();

Echo.private(`users.${currentUserId}.jobs`)
    .listen('.job.progress', (data) => {
        jobs.set(data.job_id, data);
        renderJobList(Array.from(jobs.values()));

        if (data.stage === 'completed') {
            showToast(`Export complete: ${data.job_id}`);
        }
    });
```

**Expected Output:**

- When the export job starts, the user's client receives a `job.progress` event with 0%.
- As processing continues, progress updates arrive at 5% intervals.
- The UI shows a progress bar and status text.
- On completion, a toast notification appears.

**Why This Code Produces That Result:**

- The job ID is a UUID generated in the constructor, so the client can track multiple concurrent jobs.
- Progress events fire every 5%, balancing responsiveness and overhead.
- The per-user private channel ensures only the job owner sees progress.
- The client maintains a job map keyed by job ID, so multiple jobs can be tracked independently.

### Real-World Cases

**Case 1: E-Commerce Operations Dashboard**

An operations team monitors orders/min, revenue/hour, and cart abandonment in real time. Metrics are broadcast every few seconds on a shared private channel, and the dashboard reconciles with a periodic HTTP fetch.

**Case 2: SaaS Data Import Progress**

A SaaS platform allows users to import large CSV files. The import job broadcasts progress on a per-user channel, and the UI shows a live progress bar with ETA and stage descriptions.

**Case 3: Server Telemetry Dashboard**

An infrastructure team streams CPU, memory, and request latency metrics to a NOC dashboard. Metrics are throttled to once per second, and the client keeps a rolling window for charting.

### References

- Laravel Broadcasting Documentation - https://laravel.com/docs/12.x/broadcasting
- Laravel Queues: Job Events - https://laravel.com/docs/12.x/queues#job-events
- Laravel Reverb Documentation - https://laravel.com/docs/12.x/reverb
- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation


## 4. Real-Time Status Updates (Online/Offline Status, Collaborative Presence Indicators)

### Definitions

**Core Definition:** Real-time status updates are broadcast signals that reflect a user's current state (online, offline, away, busy) or their active presence within a collaborative context, enabling UIs to show live indicators such as green dots, "last seen" timestamps, or active collaborator avatars.

**Technical Definition:** Status updates are implemented primarily via presence channels (which track join/leave events and member metadata) and via explicit user status events broadcast on private per-user channels when a user transitions between states (online, away, offline). Online/offline detection often relies on WebSocket connection lifecycle (connect/disconnect) combined with a heartbeat and a grace period to avoid flapping. Collaborative presence uses presence channel `here`/`joining`/`leaving` callbacks plus custom metadata (e.g., cursor position, current document) broadcast as throttled events.

**Beginner-Friendly Explanation:** Status updates are like the green dot next to a friend's name in a chat app. When they open the app, the dot turns green; when they close it, it turns gray. In collaborative apps, it's like seeing your teammates' cursors moving on a shared document—everyone knows who's there and what they're doing.

### Purposes

- To display online/offline/away status for users.
- To show a live list of collaborators in a shared resource.
- To indicate cursor positions or active selections in collaborative editing.
- To provide "last seen" timestamps for offline users.
- To enable presence-based features (e.g., "3 people are viewing this document").
- To reduce polling by pushing status changes to interested clients.
- To support away/idle detection based on activity or focus.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Events/UserStatusChanged.php

namespace App\Events;

use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class UserStatusChanged implements ShouldBroadcastNow
{
    public function __construct(
        public int $userId,
        public string $status,      // 'online', 'away', 'busy', 'offline'
        public ?string $lastSeen = null
    ) {}

    public function broadcastOn(): array
    {
        // Presence channel shared by all users who care about status
        return [new PresenceChannel('presence.status')];
    }

    public function broadcastAs(): string
    {
        return 'user.status.changed';
    }

    public function broadcastWith(): array
    {
        return [
            'user_id' => $this->userId,
            'status' => $this->status,
            'last_seen' => $this->lastSeen ?? now()->toIso8601String(),
        ];
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `PresenceChannel` | Tracks who is subscribed and provides join/leave |
| `UserStatusChanged` | Explicit status transition (online → away) |
| `ShouldBroadcastNow` | Low-latency status updates |
| `presence.status` | Shared presence channel for status watchers |
| `last_seen` | Timestamp for offline display |

#### Syntax Rules

1. Presence channels **must** have an authorization closure that returns an array (user metadata).
2. Join/leave events are emitted automatically by the WebSocket server when users subscribe/unsubscribe.
3. Explicit status transitions (online → away) require custom broadcast events.
4. Status events often use `ShouldBroadcastNow` for low latency.
5. Heartbeats and grace periods are typically implemented client-side and via a scheduled server task.
6. Cursor/presence metadata updates should be throttled (e.g., at most 10/sec).
7. "Last seen" timestamps should be persisted server-side for offline users.

#### Constraints and Limitations

- **Disconnect Detection Latency:** WebSocket servers detect disconnects after a timeout (often 30–60s). Presence channels may show users as online briefly after they disconnect.
- **Multiple Tabs:** A user with multiple tabs may appear multiple times in the presence list unless deduplicated by user ID.
- **Status Flapping:** Frequent connect/disconnect (e.g., unstable network) causes status flapping; a grace period smooths transitions.
- **Privacy:** Broadcasting user status globally may raise privacy concerns; scope status channels to relevant contexts (e.g., teammates only).
- **Cursor Event Volume:** High-frequency cursor updates can overwhelm the server; throttle aggressively and consider binary protocols for efficiency.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Online/Offline Status with Grace Period

**Step-by-Step Setup Guide:**

1. Create a `UserStatusChanged` event.
2. Create a presence channel `presence.status` authorized for authenticated users.
3. Track online status on connect (via Echo) and offline on disconnect (via a scheduled job or client beacon).
4. Store `last_seen_at` in the users table.
5. Display status in the UI.

**Complete Executable Code:**

```php
<?php
// database/migrations/xxxx_add_last_seen_to_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('status')->default('offline');
            $table->timestamp('last_seen_at')->nullable();
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn(['status', 'last_seen_at']);
        });
    }
};
```

```php
<?php
// app/Events/UserStatusChanged.php

namespace App\Events;

use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class UserStatusChanged implements ShouldBroadcastNow
{
    public function __construct(
        public int $userId,
        public string $status,
        public ?string $lastSeen = null
    ) {}

    public function broadcastOn(): array
    {
        return [new PresenceChannel('presence.status')];
    }

    public function broadcastAs(): string
    {
        return 'user.status.changed';
    }

    public function broadcastWith(): array
    {
        return [
            'user_id' => $this->userId,
            'status' => $this->status,
            'last_seen' => $this->lastSeen,
        ];
    }
}
```

```php
// routes/channels.php

Broadcast::channel('presence.status', function ($user) {
    return [
        'id' => $user->id,
        'name' => $user->name,
        'avatar' => $user->avatar_url,
        'status' => $user->status,
    ];
});
```

```php
// app/Http/Controllers/StatusController.php

namespace App\Http\Controllers;

use App\Events\UserStatusChanged;
use Illuminate\Http\Request;

class StatusController extends Controller
{
    public function online(Request $request)
    {
        $user = $request->user();
        $user->update(['status' => 'online', 'last_seen_at' => now()]);
        UserStatusChanged::dispatch($user->id, 'online');
        return response()->noContent();
    }

    public function away(Request $request)
    {
        $user = $request->user();
        $user->update(['status' => 'away', 'last_seen_at' => now()]);
        UserStatusChanged::dispatch($user->id, 'away');
        return response()->noContent();
    }

    public function offline(Request $request)
    {
        $user = $request->user();
        $user->update(['status' => 'offline', 'last_seen_at' => now()]);
        UserStatusChanged::dispatch($user->id, 'offline', now()->toIso8601String());
        return response()->noContent();
    }
}
```

```javascript
// resources/js/status.js

const userId = window.appConfig.userId;

// Presence channel tracks who is online
Echo.join('presence.status')
    .here((users) => {
        // Initial list of online users
        users.forEach(u => updateUserStatus(u.id, u.status || 'online'));
    })
    .joining((user) => {
        updateUserStatus(user.id, 'online');
    })
    .leaving((user) => {
        updateUserStatus(user.id, 'offline');
    })
    .listen('.user.status.changed', (data) => {
        updateUserStatus(data.user_id, data.status, data.last_seen);
    });

// Mark self online on page load
fetch('/status/online', { method: 'POST', headers: csrfHeaders() });

// Mark away when tab is hidden
document.addEventListener('visibilitychange', () => {
    const endpoint = document.hidden ? '/status/away' : '/status/online';
    fetch(endpoint, { method: 'POST', headers: csrfHeaders() });
});

// Mark offline on page unload (best effort; may not always fire)
window.addEventListener('beforeunload', () => {
    navigator.sendBeacon('/status/offline', new Blob([JSON.stringify({})], { type: 'application/json' }));
});

// Heartbeat every 30s to keep status fresh
setInterval(() => {
    fetch('/status/online', { method: 'POST', headers: csrfHeaders() });
}, 30000);
```

```php
// routes/web.php

Route::middleware('auth')->group(function () {
    Route::post('/status/online', [StatusController::class, 'online']);
    Route::post('/status/away', [StatusController::class, 'away']);
    Route::post('/status/offline', [StatusController::class, 'offline']);
});
```

**Expected Output:**

- When a user opens the app, they appear online to all subscribed clients.
- When they switch tabs, their status becomes "away."
- When they close the browser, `sendBeacon` fires `/status/offline` (best effort).
- If the beacon fails, the heartbeat stops and a scheduled job marks them offline after a grace period.
- Other users see status updates live.

**Why This Code Produces That Result:**

- Presence channels automatically emit `joining` and `leaving` events based on WebSocket subscriptions.
- Explicit status events (`online`, `away`, `offline`) handle transitions that presence alone cannot detect (e.g., tab hidden).
- The heartbeat and server-side grace period handle unreliable disconnects.
- `sendBeacon` is the most reliable way to notify the server on page unload.

#### Example 2: Collaborative Document Presence with Cursor Indicators

```php
<?php
// app/Events/CursorMoved.php

namespace App\Events;

use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class CursorMoved implements ShouldBroadcastNow
{
    public function __construct(
        public int $documentId,
        public int $userId,
        public string $userName,
        public int $position,
        public ?int $selectionEnd = null
    ) {}

    public function broadcastOn(): array
    {
        return [new PresenceChannel('documents.' . $this->documentId)];
    }

    public function broadcastAs(): string
    {
        return 'cursor.moved';
    }

    public function broadcastWith(): array
    {
        return [
            'user_id' => $this->userId,
            'user_name' => $this->userName,
            'position' => $this->position,
            'selection_end' => $this->selectionEnd,
        ];
    }
}
```

```php
// routes/channels.php

use App\Models\Document;

Broadcast::channel('documents.{documentId}', function ($user, int $documentId) {
    $document = Document::find($documentId);
    if ($document && $document->canBeEditedBy($user)) {
        return [
            'id' => $user->id,
            'name' => $user->name,
            'color' => $user->cursor_color ?? '#4f46e5',
        ];
    }
    return null;
});
```

```javascript
// Collaborative editor client

const documentId = window.docConfig.documentId;
const editor = document.getElementById('editor');

Echo.join(`documents.${documentId}`)
    .here((users) => {
        users.forEach(u => renderCursor(u.id, u.name, u.color, 0));
    })
    .joining((user) => {
        showToast(`${user.name} joined the document`);
    })
    .leaving((user) => {
        removeCursor(user.id);
    })
    .listen('.cursor.moved', (data) => {
        renderCursor(data.user_id, data.user_name, data.color, data.position);
    });

// Throttle cursor broadcasts to 10/sec
let cursorThrottle = null;
editor.addEventListener('keyup', () => {
    if (cursorThrottle) return;
    cursorThrottle = setTimeout(() => cursorThrottle = null, 100);

    const position = editor.selectionStart;
    window.axios.post(`/documents/${documentId}/cursor`, { position });
});
```

**Expected Output:**

- Each collaborator sees the other users' cursors moving in real time.
- When a user joins, a toast appears; when they leave, their cursor is removed.
- Cursor updates are throttled to 10/sec per client.

**Why This Code Produces That Result:**

- Presence channels track document collaborators.
- Cursor events are throttled to prevent flooding.
- The authorization closure checks document edit permission.
- The client renders each collaborator's cursor with their assigned color.

### Real-World Cases

**Case 1: Team Chat Online Status**

A team chat app shows green/gray dots next to each user. Presence channels track who's online, and explicit status events handle away/busy. Last-seen timestamps are shown for offline users.

**Case 2: Google Docs-Style Collaborative Editing**

A collaborative editor shows live cursors, selections, and active collaborators. Presence channels track who's in the document, and throttled cursor events broadcast positions.

**Case 3: Customer Support Agent Availability**

A support platform shows which agents are online, away, or busy. Presence channels track agent availability, and routing logic uses status to assign tickets.

### References

- Laravel Broadcasting: Presence Channels - https://laravel.com/docs/12.x/broadcasting#presence-channels
- Laravel Echo: Presence Callbacks - https://laravel.com/docs/12.x/broadcasting#joining-presence-channels
- MDN: Navigator.sendBeacon() - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon
- WebSocket Protocol (RFC 6455) - https://datatracker.ietf.org/doc/html/rfc6455


## 5. Enhanced: Broadcast Tagging and Conditions (`broadcastWhen` for Conditional Real-Time Actions)

### Definitions

**Core Definition:** `broadcastWhen()` is a method on a broadcast event that determines whether the event should actually be broadcast, allowing developers to gate real-time delivery based on model state, thresholds, user roles, or arbitrary conditions.

**Technical Definition:** When Laravel's `BroadcastEvent` job is processed, it invokes the event's `broadcastWhen()` method (if defined) before broadcasting. If the method returns `false`, the broadcast is skipped entirely—no payload is delivered, no channel subscription is triggered, and no network overhead is incurred. The method receives no arguments and must return a boolean. It is evaluated on the queue worker (for `ShouldBroadcast`) or in-process (for `ShouldBroadcastNow`).

**Beginner-Friendly Explanation:** `broadcastWhen` is like a bouncer at a club door. Even if you've prepared the event, the bouncer checks: "Should this go out?" If the answer is no (e.g., the stock level is fine, the user is already online), the event is silently dropped. It's a filter that prevents noise.

### Purposes

- To prevent broadcasting events that would be noise (e.g., minor metric changes).
- To gate real-time actions on model state (e.g., only broadcast when status changed).
- To respect user preferences (e.g., "do not disturb" mode).
- To enforce thresholds (e.g., only broadcast alerts above a severity level).
- To conditionally broadcast based on time of day or business hours.
- To reduce queue and network load by skipping unnecessary broadcasts.
- To implement feature-flagged broadcasting (only broadcast if feature enabled).

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// app/Events/InventoryLevelChanged.php

namespace App\Events;

use App\Models\Product;
use Illuminate\Broadcasting\Channel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class InventoryLevelChanged implements ShouldBroadcast
{
    public function __construct(
        public Product $product,
        public int $previousQuantity,
        public int $newQuantity
    ) {}

    public function broadcastOn(): array
    {
        return [new Channel('inventory.' . $this->product->id)];
    }

    public function broadcastAs(): string
    {
        return 'inventory.changed';
    }

    /**
     * Only broadcast if the inventory level crossed a threshold
     * or changed significantly. This prevents broadcasting
     * every minor adjustment.
     */
    public function broadcastWhen(): bool
    {
        // Always broadcast if crossing the low-stock threshold
        if ($this->previousQuantity > $this->product->low_stock_threshold
            && $this->newQuantity <= $this->product->low_stock_threshold) {
            return true;
        }

        // Always broadcast if out of stock
        if ($this->newQuantity === 0) {
            return true;
        }

        // Otherwise, only broadcast if change is >= 10%
        $changePercent = abs(($this->newQuantity - $this->previousQuantity)
            / max(1, $this->previousQuantity)) * 100;

        return $changePercent >= 10;
    }

    public function broadcastWith(): array
    {
        return [
            'product_id' => $this->product->id,
            'previous_quantity' => $this->previousQuantity,
            'new_quantity' => $this->newQuantity,
            'threshold_crossed' => $this->previousQuantity > $this->product->low_stock_threshold
                && $this->newQuantity <= $this->product->low_stock_threshold,
            'updated_at' => now()->toIso8601String(),
        ];
    }
}
```

**Component Breakdown:**

| Component | Purpose |
|-----------|---------|
| `broadcastWhen(): bool` | Returns true to broadcast, false to skip |
| Model state checks | Gate on business logic |
| Threshold comparisons | Prevent noise for minor changes |
| Time/role checks | Gate on context |
| Evaluated on queue worker | For `ShouldBroadcast` |

#### Syntax Rules

1. `broadcastWhen()` **must** return a boolean.
2. It receives **no arguments**; all data must come from the event's properties.
3. It is evaluated **before** the broadcast is sent; if `false`, nothing is delivered.
4. For `ShouldBroadcast` events, `broadcastWhen()` runs on the queue worker, not during dispatch.
5. For `ShouldBroadcastNow` events, it runs in-process during dispatch.
6. It can be combined with `broadcastWith()` for payload customization.
7. It is **not** a replacement for channel authorization; it gates the broadcast, not the subscription.

#### Constraints and Limitations

- **No Per-Listener Gating:** `broadcastWhen()` gates the entire broadcast, not per-channel or per-listener. All subscribers either receive it or none do.
- **Queue Worker Dependency:** For `ShouldBroadcast`, the condition is evaluated when the queue worker processes the job, not when the event is dispatched. Model state may have changed by then.
- **No Side Effects:** `broadcastWhen()` should be a pure predicate; avoid side effects (logging is fine, but don't mutate state).
- **Debugging Difficulty:** Skipped broadcasts are silent; add logging if you need to audit why a broadcast was skipped.
- **Not for Authorization:** Use channel authorization for access control; use `broadcastWhen` for noise reduction and business rules.

### Multiple Annotated Step-by-Step Code Examples

#### Example 1: Threshold-Based Alert Broadcasting

**Step-by-Step Setup Guide:**

1. Create a `ServerHealthAlert` event.
2. Implement `broadcastWhen()` to only broadcast when CPU > 80% or memory > 90%.
3. Dispatch the event on every health check.
4. Verify that low-severity checks are silently skipped.

**Complete Executable Code:**

```php
<?php
// app/Events/ServerHealthAlert.php

namespace App\Events;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerHealthAlert implements ShouldBroadcast
{
    public function __construct(
        public string $serverId,
        public float $cpuPercent,
        public float $memoryPercent,
        public float $diskPercent
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('ops.alerts')];
    }

    public function broadcastAs(): string
    {
        return 'server.health.alert';
    }

    /**
     * Only broadcast when a threshold is breached.
     * Normal health checks are silently dropped, preventing
     * noise on the ops dashboard.
     */
    public function broadcastWhen(): bool
    {
        $cpuBreached = $this->cpuPercent > 80;
        $memoryBreached = $this->memoryPercent > 90;
        $diskBreached = $this->diskPercent > 85;

        $breached = $cpuBreached || $memoryBreached || $diskBreached;

        // Log the evaluation for auditing (optional)
        if (!$breached) {
            \Log::debug('Health alert skipped', [
                'server' => $this->serverId,
                'cpu' => $this->cpuPercent,
                'memory' => $this->memoryPercent,
                'disk' => $this->diskPercent,
            ]);
        }

        return $breached;
    }

    public function broadcastWith(): array
    {
        return [
            'server_id' => $this->serverId,
            'cpu' => $this->cpuPercent,
            'memory' => $this->memoryPercent,
            'disk' => $this->diskPercent,
            'severity' => $this->severity(),
            'recorded_at' => now()->toIso8601String(),
        ];
    }

    private function severity(): string
    {
        if ($this->cpuPercent > 95 || $this->memoryPercent > 95) {
            return 'critical';
        }
        if ($this->cpuPercent > 80 || $this->memoryPercent > 90) {
            return 'warning';
        }
        return 'info';
    }
}
```

```php
// app/Console/Commands/CheckServerHealth.php

namespace App\Console\Commands;

use App\Events\ServerHealthAlert;
use Illuminate\Console\Command;

class CheckServerHealth extends Command
{
    protected $signature = 'health:check';
    protected $description = 'Check server health and broadcast alerts if thresholds breached';

    public function handle(): void
    {
        // Simulate fetching health metrics
        $metrics = [
            'server_id' => 'web-01',
            'cpu_percent' => 45.2,
            'memory_percent' => 62.1,
            'disk_percent' => 55.0,
        ];

        // This dispatch will be silently skipped because no threshold is breached
        ServerHealthAlert::dispatch(
            $metrics['server_id'],
            $metrics['cpu_percent'],
            $metrics['memory_percent'],
            $metrics['disk_percent']
        );

        $this->info('Health check dispatched (may be skipped by broadcastWhen)');
    }
}
```

**Expected Output:**

- Running `php artisan health:check` dispatches the event.
- Because no threshold is breached, `broadcastWhen()` returns `false`, and no broadcast occurs.
- A debug log entry records the skipped broadcast.
- If CPU were 85%, the broadcast would proceed with severity `warning`.

**Why This Code Produces That Result:**

- `broadcastWhen()` evaluates thresholds and returns `false` when all metrics are normal.
- The queue worker (or in-process for `ShouldBroadcastNow`) skips the broadcast entirely.
- Logging provides visibility into skipped broadcasts for auditing.

#### Example 2: Conditional Broadcasting Based on User Preferences

```php
<?php
// app/Events/TaskCommented.php

namespace App\Events;

use App\Models\Task;
use App\Models\User;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class TaskCommented implements ShouldBroadcast
{
    public function __construct(
        public Task $task,
        public User $commenter,
        public User $recipient,
        public string $commentBody
    ) {}

    public function broadcastOn(): array
    {
        return [new PrivateChannel('users.' . $this->recipient->id)];
    }

    public function broadcastAs(): string
    {
        return 'task.commented';
    }

    /**
     * Respect the recipient's notification preferences:
     * - Do not broadcast if the recipient has muted this task
     * - Do not broadcast during the recipient's "do not disturb" hours
     * - Do not broadcast if the commenter is the recipient
     */
    public function broadcastWhen(): bool
    {
        // Don't notify the commenter about their own comment
        if ($this->commenter->id === $this->recipient->id) {
            return false;
        }

        // Respect task-level mute
        if ($this->recipient->hasMutedTask($this->task->id)) {
            return false;
        }

        // Respect do-not-disturb hours
        if ($this->recipient->isInDoNotDisturbWindow()) {
            return false;
        }

        return true;
    }

    public function broadcastWith(): array
    {
        return [
            'task_id' => $this->task->id,
            'task_title' => $this->task->title,
            'commenter_name' => $this->commenter->name,
            'comment_preview' => \Str::limit($this->commentBody, 80),
            'commented_at' => now()->toIso8601String(),
        ];
    }
}
```

```php
// app/Models/User.php (excerpt)

public function hasMutedTask(int $taskId): bool
{
    return $this->mutedTasks()->where('task_id', $taskId)->exists();
}

public function isInDoNotDisturbWindow(): bool
{
    if (!$this->dnd_start || !$this->dnd_end) {
        return false;
    }
    $now = now()->format('H:i');
    return $now >= $this->dnd_start && $now <= $this->dnd_end;
}
```

**Expected Output:**

- If the recipient has muted the task, no broadcast occurs.
- If the recipient is in their DND window, no broadcast occurs.
- If the commenter is the recipient, no broadcast occurs.
- Otherwise, the broadcast proceeds and the recipient sees a live comment notification.

**Why This Code Produces That Result:**

- `broadcastWhen()` encapsulates all gating logic in one place.
- The event is still dispatched (and could be persisted via a notification), but the real-time broadcast is conditionally skipped.
- This respects user preferences without requiring the dispatch site to know about them.

### Real-World Cases

**Case 1: Stock Trading Alerts**

A trading platform broadcasts price alerts only when a stock crosses a user-defined threshold. `broadcastWhen()` checks the threshold, preventing alerts for every minor price tick.

**Case 2: IoT Sensor Monitoring**

An IoT platform broadcasts sensor alerts only when readings exceed safe ranges. `broadcastWhen()` gates on thresholds, reducing dashboard noise and preserving bandwidth.

**Case 3: Social Media Engagement Notifications**

A social platform broadcasts engagement notifications only for users who haven't muted the post and aren't in DND hours. `broadcastWhen()` enforces these preferences.

### References

- Laravel Broadcasting: Conditional Broadcasting - https://laravel.com/docs/12.x/broadcasting#conditionally-broadcasting-events
- Laravel Broadcasting: Broadcast Events - https://laravel.com/docs/12.x/broadcasting#defining-broadcast-events
- Laravel API: ShouldBroadcast - https://api.laravel.com/docs/11.x/Illuminate/Contracts/Broadcasting/ShouldBroadcast.html
- Laravel Notifications: Broadcast Notifications - https://laravel.com/docs/12.x/notifications#broadcast-notifications


## Summary Table of Real-Time Features

| Feature | Primary Channel Type | Key Events/Methods | Typical Queue Strategy |
|---------|---------------------|-------------------|----------------------|
| Notifications | Private (`App.Models.User.{id}`) | `toBroadcast()`, `via('broadcast')` | Queued (`ShouldQueue`) |
| Chat Systems | Private (`conversations.{id}`) + Presence | `MessageSent`, `MessageStateChanged`, `UserTyping` | Queued for messages, `ShouldBroadcastNow` for typing |
| Live Dashboards | Private (`dashboards.metrics`) | `MetricUpdated`, `JobProgressUpdated` | Queued with throttling |
| Status Updates | Presence (`presence.status`, `documents.{id}`) | `UserStatusChanged`, `CursorMoved` | `ShouldBroadcastNow` for low latency |
| Conditional Broadcasting | Any | `broadcastWhen()` | Evaluated on queue worker |

---

## References

- Laravel Broadcasting Documentation (12.x) - https://laravel.com/docs/12.x/broadcasting
- Laravel Notifications Documentation (12.x) - https://laravel.com/docs/12.x/notifications
- Laravel Notifications: Broadcast Notifications - https://laravel.com/docs/12.x/notifications#broadcast-notifications
- Laravel Broadcasting: Private Channels - https://laravel.com/docs/12.x/broadcasting#private-channels
- Laravel Broadcasting: Presence Channels - https://laravel.com/docs/12.x/broadcasting#presence-channels
- Laravel Broadcasting: Conditional Broadcasting - https://laravel.com/docs/12.x/broadcasting#conditionally-broadcasting-events
- Laravel Echo Documentation - https://laravel.com/docs/12.x/broadcasting#client-side-installation
- Laravel Reverb Documentation - https://laravel.com/docs/12.x/reverb
- Laravel Queues Documentation - https://laravel.com/docs/12.x/queues
- Laravel API: BroadcastMessage - https://api.laravel.com/docs/11.x/Illuminate/Notifications/Messages/BroadcastMessage.html
- Laravel API: ShouldBroadcast - https://api.laravel.com/docs/11.x/Illuminate/Contracts/Broadcasting/ShouldBroadcast.html
- MDN: Navigator.sendBeacon() - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon
- WebSocket Protocol (RFC 6455) - https://datatracker.ietf.org/doc/html/rfc6455
- Pusher Channels Documentation - https://pusher.com/docs/channels/
- Ably Documentation - https://ably.com/docs