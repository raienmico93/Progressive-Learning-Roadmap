# Laravel Notification Engine — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Laravel Notification Engine is the framework's subsystem for sending short, informational messages to users across multiple delivery channels — including email, SMS, Slack, database, and broadcast — through a unified, channel-agnostic API.

**Technical Definition:** Laravel's notification system is built around the `Illuminate\Notifications\Notification` base class, the `Illuminate\Notifications\ChannelManager` (which resolves channel drivers), and the `Illuminate\Notifications\NotificationSender` (which orchestrates delivery). Each notification is a class extending `Notification` that defines a `via()` method returning an array of channel names, and one message-building method per channel (`toMail()`, `toDatabase()`, `toBroadcast()`, `toVonage()`, `toSlack()`, etc.). The `Notifiable` trait, applied to Eloquent models, provides the `notify()` and `notifyNow()` methods. The `Notification` facade provides `send()`, `sendNow()`, and the `route()` method for on-demand notifications. Channels are resolved by the `ChannelManager` from drivers registered in `AppServiceProvider` or auto-discovered from the `Illuminate\Notifications\Channels` namespace.

**Beginner-Friendly Explanation:** Laravel's notification system lets you send messages to users through many different channels — email, SMS, Slack, your app's database, or real-time browser updates — all from a single PHP class. You write one notification class that says "send this via email and database," and Laravel handles the details of each channel. It's like writing one message and having Laravel post it to all the right places.

### Key Characteristics

- **Single class, multiple channels:** One notification class can deliver via email, SMS, Slack, database, broadcast, and custom channels simultaneously.
- **Channel-agnostic API:** The same `notify()` method works regardless of which channels are used.
- **Queue support:** Notifications can implement `ShouldQueue` for background delivery, preventing them from blocking HTTP requests.
- **On-demand delivery:** Notifications can be sent to non-user endpoints (email addresses, Slack webhooks, phone numbers) via `Notification::route()`.
- **Localization support:** Notification content can be automatically translated to the recipient's preferred locale.
- **Custom channel extensibility:** Developers can create custom channel drivers by implementing a class with a `send()` method.
- **Database storage:** Notifications can be persisted in a `notifications` table for in-app display.
- **Real-time broadcasting:** Notifications can be broadcast over WebSockets using Pusher or Laravel Reverb.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with the `Notifiable` trait on the `User` model (included by default).
- For database notifications: the `notifications` table migration.
- For broadcast notifications: a configured broadcasting driver (Pusher, Reverb, Ably, etc.) and Laravel Echo on the frontend.
- For SMS notifications: a Vonage (Nexmo) account and the `laravel/vonage-notification-channel` package.
- For Slack notifications: a Slack webhook URL and the `laravel/slack-notification-channel` package.

### Related Programming Areas

- **Mail System** — The mail channel builds on Laravel's mailable infrastructure.
- **Queue System** — Notifications implementing `ShouldQueue` are dispatched to the queue.
- **Broadcasting** — The broadcast channel uses Laravel's event broadcasting system.
- **Database** — The database channel stores notifications in a `notifications` table.
- **Localization** — Notification content can be translated based on the recipient's locale.
- **Service Container** — Custom channels are registered and resolved through the container.

### Core Concepts / Features

1. Notification Classes (Structuring multi-channel delivery schemas via the central `via()` hub method)
2. Core Internal Channels (Configuring built-in Mail, Database, and real-time Pusher/Reverb Broadcast notifications)
3. On-Demand Notifications (Utilizing the `Notification::route` facade to send alerts to non-user or anonymous endpoints)
4. Localization (Adapting email templates dynamically to user-specific locales)
5. Custom Channels (Constructing bespoke delivery channel gateways)

---

## 1. Notification Classes

### Definitions

**Core Definition:** A notification class is a PHP class extending `Illuminate\Notifications\Notification` that defines which channels a notification uses and how it is formatted for each channel.

**Technical Definition:** The `Notification` base class provides the `via()` method, which receives the notifiable entity and returns an array of channel driver names. Each channel name corresponds to a method on the notification class that builds the channel-specific message (e.g., `toMail()` for the `mail` channel, `toDatabase()` for the `database` channel). The `NotificationSender` iterates over the channels returned by `via()`, resolves each channel driver from the `ChannelManager`, calls the appropriate message-building method, and delegates delivery to the driver's `send()` method. Notification classes are generated via `php artisan make:notification` and stored in `app/Notifications`.

**Beginner-Friendly Explanation:** A notification class is a single PHP file that describes one type of notification your app sends — like "Invoice Paid" or "Order Shipped." Inside, you list which channels it should use (email, database, SMS), and for each channel you write a small method that formats the message. When you send the notification, Laravel reads your channel list and delivers the formatted message to each channel.

### Purposes

- To encapsulate the definition, channels, and formatting of a single notification type in a dedicated class.
- To enable a single notification to be delivered across multiple channels simultaneously.
- To separate notification logic from the code that triggers it.
- To support queuing for background delivery via the `ShouldQueue` interface.
- To provide a consistent, testable interface for all application notifications.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Notifications/InvoicePaid.php

namespace App\Notifications;

use App\Models\Invoice;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Messages\BroadcastMessage;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Invoice $invoice,
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail', 'database', 'broadcast'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Invoice Paid')
            ->line('Your invoice has been paid.')
            ->action('View Invoice', url('/invoices/' . $this->invoice->id))
            ->line('Thank you for your business!');
    }

    public function toDatabase(object $notifiable): array
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount'     => $this->invoice->amount,
            'message'    => 'Invoice #' . $this->invoice->number . ' has been paid.',
        ];
    }

    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount'     => $this->invoice->amount,
            'message'    => 'Invoice paid.',
        ]);
    }
}
```

**Component Breakdown:**

- `via($notifiable)` — Returns an array of channel names. The `$notifiable` argument allows conditional channel selection based on the recipient.
- `toMail($notifiable)` — Builds the email message. Returns a `MailMessage` instance.
- `toDatabase($notifiable)` — Returns an array that is JSON-encoded and stored in the `notifications` table.
- `toBroadcast($notifiable)` — Returns a `BroadcastMessage` for real-time delivery.
- `ShouldQueue` — When implemented, the notification is queued for background delivery.

**Syntax Rules:**

- The `via()` method must return an array of channel names, even if only one channel is used.
- Each channel in the `via()` array must have a corresponding `to{Channel}()` method (e.g., `mail` → `toMail()`, `database` → `toDatabase()`).
- The `$notifiable` argument is the entity receiving the notification (typically a `User` model instance).
- Notifications can be sent via `$user->notify(new InvoicePaid($invoice))` or `Notification::send($users, new InvoicePaid($invoice))`.
- The `via()` method can use the `$notifiable` to conditionally include channels (e.g., only send SMS if the user has a phone number).

**Constraints and Limitations:**

- **The `toDatabase()` or `toArray()` method is required for the database channel.** Without it, database delivery fails.
- **The `toBroadcast()` method requires a configured broadcasting driver.** Without one, broadcast delivery fails.
- **The `toMail()` method requires mail configuration.** The `log` driver can be used for local testing.
- **Notifications implementing `ShouldQueue` are queued, not sent immediately.** Use `Notification::sendNow()` to bypass the queue.
- **The `via()` method cannot return an empty array.** At least one channel must be specified.

### Annotated Code Examples

**Example 1: Multi-Channel Notification with Conditional Channels**

```php
<?php
// File: app/Notifications/OrderShipped.php

namespace App\Notifications;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Messages\BroadcastMessage;

class OrderShipped extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order,
    ) {}

    public function via(object $notifiable): array
    {
        // Step 1: Determine channels based on the recipient's preferences
        $channels = ['mail', 'database', 'broadcast'];

        // Step 2: Only add SMS if the user has a phone number
        if ($notifiable->phone_number) {
            $channels[] = 'vonage';
        }

        return $channels;
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Order #' . $this->order->number . ' Shipped')
            ->greeting('Hello ' . $notifiable->name . '!')
            ->line('Your order has been shipped.')
            ->line('Tracking number: ' . $this->order->tracking_number)
            ->action('Track Order', url('/orders/' . $this->order->id))
            ->line('Thank you for shopping with us!');
    }

    public function toDatabase(object $notifiable): array
    {
        return [
            'order_id'       => $this->order->id,
            'order_number'   => $this->order->number,
            'tracking_number' => $this->order->tracking_number,
            'message'        => 'Order #' . $this->order->number . ' has been shipped.',
        ];
    }

    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'order_id'       => $this->order->id,
            'order_number'   => $this->order->number,
            'message'        => 'Order shipped.',
        ]);
    }

    public function toVonage(object $notifiable): \Illuminate\Notifications\Messages\VonageMessage
    {
        return (new \Illuminate\Notifications\Messages\VonageMessage)
            ->content('Your order #' . $this->order->number . ' has been shipped.');
    }
}
```

```php
// Sending the notification
use App\Notifications\OrderShipped;

$order = Order::find(123);
$order->customer->notify(new OrderShipped($order));
```

**Expected Output:**

- The customer receives an email with the subject "Order #12345 Shipped."
- A database record is created in the `notifications` table.
- A broadcast event is dispatched to the customer's private channel.
- If the customer has a phone number, an SMS is sent via Vonage.

**Why This Output Occurs:** The `via()` method checks the `$notifiable` (the customer) for a phone number and conditionally includes the `vonage` channel. Each channel's message-building method formats the notification appropriately. The `NotificationSender` iterates over the channels, resolves each driver, and delivers the message. Because the notification implements `ShouldQueue`, the entire process runs in the background.

---

**Example 2: Database-Only Notification with toArray**

```php
<?php
// File: app/Notifications/NewMessage.php

namespace App\Notifications;

use App\Models\Message;
use Illuminate\Notifications\Notification;

class NewMessage extends Notification
{
    public function __construct(
        public Message $message,
    ) {}

    public function via(object $notifiable): array
    {
        return ['database'];
    }

    public function toArray(object $notifiable): array
    {
        return [
            'message_id'  => $this->message->id,
            'sender_name' => $this->message->sender->name,
            'preview'     => \Str::limit($this->message->body, 100),
            'url'         => url('/messages/' . $this->message->id),
        ];
    }
}
```

```php
// Sending the notification
$user->notify(new NewMessage($message));

// Retrieving unread notifications
foreach ($user->unreadNotifications as $notification) {
    echo $notification->data['preview'];
}
```

**Expected Output:** A record is inserted into the `notifications` table with the JSON-encoded data. The notification appears in the user's unread notifications list.

**Why This Output Occurs:** The `toArray()` method returns the data array, which Laravel JSON-encodes and stores in the `data` column of the `notifications` table. The `unreadNotifications` relationship retrieves records where `read_at` is `null`. This pattern is ideal for in-app notification feeds.

### Real-World Cases

- **E-commerce platforms:** An `OrderShipped` notification sends an email with tracking details, stores a database record for the user's notification feed, and broadcasts a real-time update to the browser.
- **SaaS applications:** An `InvoicePaid` notification emails the receipt, records the payment in the database, and sends an SMS confirmation.
- **Social networks:** A `NewFollower` notification stores a database record and broadcasts a real-time toast to the user's browser.
- **Healthcare systems:** An `AppointmentReminder` notification sends an email and SMS 24 hours before the appointment, and stores a database record for audit purposes.
- **Project management tools:** A `TaskAssigned` notification emails the assignee, stores a database record, and broadcasts a real-time update to the assignee's dashboard.

---

## 2. Core Internal Channels

### Definitions

**Core Definition:** Core internal channels are the built-in notification delivery drivers provided by Laravel — Mail, Database, and Broadcast — that handle the most common notification delivery methods without additional packages.

**Technical Definition:** The Mail channel (`Illuminate\Notifications\Channels\MailChannel`) converts the `MailMessage` returned by `toMail()` into a mailable and sends it via the mail system. The Database channel (`Illuminate\Notifications\Channels\DatabaseChannel`) stores the array returned by `toDatabase()` or `toArray()` in the `notifications` table, JSON-encoded in the `data` column. The Broadcast channel (`Illuminate\Notifications\Channels\BroadcastChannel`) dispatches a `BroadcastNotificationCreated` event to the configured broadcasting driver (Pusher, Reverb, Ably, etc.), enabling real-time delivery to connected clients.

**Beginner-Friendly Explanation:** Laravel comes with three built-in ways to send notifications: email (via your mail system), database (stored in a table for in-app display), and broadcast (sent instantly to the user's browser via WebSockets). You list these channel names in your notification's `via()` method, and Laravel handles the delivery for you.

### Purposes

- To send notification emails using Laravel's mailable infrastructure and templates.
- To persist notifications in the database for in-app display and retrieval.
- To broadcast notifications in real time to connected clients via WebSockets.
- To combine multiple channels in a single notification for comprehensive user engagement.
- To provide a consistent API across email, database, and real-time delivery.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Channel names in via()
public function via(object $notifiable): array
{
    return ['mail', 'database', 'broadcast'];
}

// Mail channel: toMail()
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Notification Subject')
        ->greeting('Hello!')
        ->line('First line of content.')
        ->action('Button Text', url('/path'))
        ->line('Closing line.');
}

// Database channel: toDatabase() or toArray()
public function toDatabase(object $notifiable): array
{
    return [
        'key'   => 'value',
        'url'   => url('/path'),
    ];
}

// Broadcast channel: toBroadcast()
public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'key'   => 'value',
        'url'   => url('/path'),
    ]);
}
```

**Component Breakdown:**

- `'mail'` — The mail channel. Requires `toMail()` returning a `MailMessage`.
- `'database'` — The database channel. Requires `toDatabase()` or `toArray()` returning an array.
- `'broadcast'` — The broadcast channel. Requires `toBroadcast()` returning a `BroadcastMessage`.
- `MailMessage` — Provides methods for building email content: `subject()`, `greeting()`, `line()`, `action()`, `salutation()`, `level()`, `error()`.
- `BroadcastMessage` — Wraps an array of data for broadcasting.
- `database` table — The `notifications` table stores the `id`, `type`, `notifiable_type`, `notifiable_id`, `data` (JSON), `read_at`, and `created_at` columns.

**Syntax Rules:**

- The `mail` channel requires mail configuration in `config/mail.php`; the `log` driver is useful for local development.
- The `database` channel requires the `notifications` table migration (`php artisan notifications:table && php artisan migrate`).
- The `broadcast` channel requires a broadcasting driver configured in `config/broadcasting.php` (Pusher, Reverb, Ably, etc.) and Laravel Echo on the frontend.
- The `database` channel automatically uses `toArray()` if `toDatabase()` is not defined.
- The `broadcast` channel automatically uses `toArray()` if `toBroadcast()` is not defined.

**Constraints and Limitations:**

- **The mail channel requires a configured mailer.** Without it, the notification fails.
- **The database channel requires the `notifications` table.** Run the migration before using it.
- **The broadcast channel requires a running WebSocket server.** Without one, broadcasts are not delivered.
- **The database channel stores the `data` column as JSON.** Complex objects must be serialisable.
- **The broadcast channel sends an event to the `private-App.Models.User.{id}` channel by default.** The frontend must subscribe to this channel.

### Annotated Code Examples

**Example 1: Complete Multi-Channel Notification with Mail, Database, and Broadcast**

```php
<?php
// File: app/Notifications/DeploymentCompleted.php

namespace App\Notifications;

use App\Models\Deployment;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Messages\BroadcastMessage;

class DeploymentCompleted extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Deployment $deployment,
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail', 'database', 'broadcast'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Deployment #' . $this->deployment->id . ' Completed')
            ->greeting('Hello ' . $notifiable->name . '!')
            ->line('Your deployment has completed successfully.')
            ->line('Environment: ' . $this->deployment->environment)
            ->line('Duration: ' . $this->deployment->duration . ' seconds')
            ->action('View Deployment', url('/deployments/' . $this->deployment->id))
            ->salutation('— The ' . config('app.name') . ' Team');
    }

    public function toDatabase(object $notifiable): array
    {
        return [
            'deployment_id' => $this->deployment->id,
            'environment'   => $this->deployment->environment,
            'duration'      => $this->deployment->duration,
            'message'       => 'Deployment #' . $this->deployment->id . ' completed.',
            'url'           => url('/deployments/' . $this->deployment->id),
        ];
    }

    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'deployment_id' => $this->deployment->id,
            'environment'   => $this->deployment->environment,
            'message'       => 'Deployment completed.',
        ]);
    }
}
```

```php
// Sending the notification
use App\Notifications\DeploymentCompleted;

$deployment = Deployment::find(456);
$user->notify(new DeploymentCompleted($deployment));
```

**Expected Output:**

- An email is sent with the subject "Deployment #456 Completed" and a "View Deployment" button.
- A database record is created with the deployment details.
- A broadcast event is dispatched to the user's private channel, triggering a real-time toast notification.

**Why This Output Occurs:** The `via()` method returns `['mail', 'database', 'broadcast']`, so all three channels are used. The `toMail()` method builds a formatted email with a subject, greeting, lines, and action button. The `toDatabase()` method returns an array that is JSON-encoded and stored in the `notifications` table. The `toBroadcast()` method returns a `BroadcastMessage` that is dispatched via the configured broadcasting driver.

---

**Example 2: Database Notification with Unread Retrieval**

```php
<?php
// File: app/Http/Controllers/NotificationController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class NotificationController extends Controller
{
    public function index(Request $request)
    {
        // Step 1: Retrieve unread notifications
        $notifications = $request->user()->unreadNotifications;

        return response()->json([
            'count'         => $notifications->count(),
            'notifications' => $notifications->map(function ($notification) {
                return [
                    'id'      => $notification->id,
                    'data'    => $notification->data,
                    'read_at' => $notification->read_at,
                ];
            }),
        ]);
    }

    public function markAsRead(Request $request, string $id)
    {
        // Step 2: Mark a single notification as read
        $notification = $request->user()->notifications()->findOrFail($id);
        $notification->markAsRead();

        return response()->json(['message' => 'Notification marked as read.']);
    }

    public function markAllAsRead(Request $request)
    {
        // Step 3: Mark all notifications as read
        $request->user()->unreadNotifications->markAsRead();

        return response()->json(['message' => 'All notifications marked as read.']);
    }
}
```

**Expected Output:**

- The `index` endpoint returns the unread notifications with their data and read status.
- The `markAsRead` endpoint marks a single notification as read.
- The `markAllAsRead` endpoint marks all notifications as read.

**Why This Output Occurs:** The `unreadNotifications` relationship retrieves notifications where `read_at` is `null`. The `markAsRead()` method updates the `read_at` column to the current timestamp. The `markAsRead()` method on the collection updates all unread notifications in a single query. These methods are provided by the `HasDatabaseNotifications` trait.

### Real-World Cases

- **Deployment dashboards:** A `DeploymentCompleted` notification emails the team, stores a database record for the activity feed, and broadcasts a real-time update to the dashboard.
- **E-commerce order tracking:** An `OrderShipped` notification emails the customer, stores a database record for the notification bell, and broadcasts a real-time update.
- **Social media engagement:** A `NewLike` notification stores a database record and broadcasts a real-time toast to the user's browser.
- **Customer support:** A `TicketReplied` notification emails the customer and stores a database record for the support agent's dashboard.
- **Financial alerts:** A `PaymentReceived` notification emails the recipient, stores a database record, and broadcasts a real-time notification to the accounting team.

---

## 3. On-Demand Notifications

### Definitions

**Core Definition:** On-demand notifications are notifications sent to recipients who are not stored as users in the application — such as email addresses, Slack webhooks, or phone numbers — using the `Notification::route()` method to specify ad-hoc routing information.

**Technical Definition:** The `Notification` facade's `route()` method returns an `AnonymousNotifiable` instance, which allows chaining multiple `route()` calls to specify delivery endpoints for different channels. When the notification is sent, the `AnonymousNotifiable` is passed to the `via()` method as the `$notifiable` argument, and the channel drivers retrieve routing information via the `routeNotificationFor{Channel}()` method (e.g., `routeNotificationForMail()`). This enables notifications to be sent to recipients who have no user model or database presence.

**Beginner-Friendly Explanation:** Sometimes you need to send a notification to someone who isn't a user of your application — like a customer who placed a guest order, or an external stakeholder who needs a one-time alert. On-demand notifications let you specify the email address, phone number, or Slack webhook directly, without creating a user account first.

### Purposes

- To send notifications to recipients who are not registered users of the application.
- To deliver alerts to external endpoints (email addresses, Slack webhooks, phone numbers) without creating user models.
- To enable guest checkout order confirmations and shipping notifications.
- To send one-time alerts to stakeholders, vendors, or emergency contacts.
- To route notifications to multiple endpoints for the same recipient across different channels.

### Syntax Rules and Structure

#### Complete General Syntax

```php
use Illuminate\Support\Facades\Notification;

// Single channel
Notification::route('mail', 'taylor@example.com')
    ->notify(new InvoicePaid($invoice));

// Multiple channels
Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', 'https://hooks.slack.com/services/...')
    ->notify(new TradeSuccessful($trade));

// Sending to a specific channel only
Notification::route('mail', 'admin@example.com')
    ->notify(new SystemAlert($alert));
```

**Component Breakdown:**

- `Notification::route($channel, $route)` — Specifies the routing information for a channel. Returns an `AnonymousNotifiable` instance.
- `$channel` — The channel name (`mail`, `vonage`, `slack`, etc.).
- `$route` — The routing information: an email address, phone number, Slack webhook URL, etc.
- `->notify($notification)` — Sends the notification to the anonymous notifiable.

**Syntax Rules:**

- Each `route()` call specifies the routing information for one channel.
- The `via()` method of the notification must include the channels specified in the `route()` calls.
- The `$notifiable` argument in `toMail()`, `toDatabase()`, etc., will be an `AnonymousNotifiable` instance.
- The `AnonymousNotifiable` provides a `routeNotificationFor()` method that retrieves the routing information for a given channel.
- On-demand notifications cannot use the `database` channel because there is no user model to associate the notification with.

**Constraints and Limitations:**

- **The `database` channel is not available for on-demand notifications.** There is no notifiable entity to associate the record with.
- **The `route()` method must be called for every channel in `via()`.** If a channel has no route, the notification fails for that channel.
- **The `AnonymousNotifiable` instance is not persisted.** It exists only for the duration of the notification send.
- **The `toDatabase()` method is not called for on-demand notifications.** Use `toArray()` if needed for other channels.

### Annotated Code Examples

**Example 1: Sending an On-Demand Notification to Multiple Channels**

```php
<?php
// File: app/Notifications/OrderConfirmation.php

namespace App\Notifications;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Messages\VonageMessage;

class OrderConfirmation extends Notification
{
    use Queueable;

    public function __construct(
        public Order $order,
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail', 'vonage'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Order Confirmation #' . $this->order->number)
            ->greeting('Thank you for your order!')
            ->line('Your order #' . $this->order->number . ' has been received.')
            ->line('Total: $' . number_format($this->order->total, 2))
            ->action('View Order', url('/orders/' . $this->order->id))
            ->line('We will notify you when your order ships.');
    }

    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
            ->content('Your order #' . $this->order->number . ' has been received. Total: $' . number_format($this->order->total, 2));
    }
}
```

```php
// Sending to a guest customer via email and SMS
use App\Notifications\OrderConfirmation;
use Illuminate\Support\Facades\Notification;

$order = Order::find(789);

Notification::route('mail', 'guest@example.com')
    ->route('vonage', '15551234567')
    ->notify(new OrderConfirmation($order));
```

**Expected Output:**

- The guest customer receives an email with the order confirmation.
- The guest customer receives an SMS with the order confirmation.

**Why This Output Occurs:** The `Notification::route()` method creates an `AnonymousNotifiable` with routing information for the `mail` and `vonage` channels. The `via()` method returns `['mail', 'vonage']`, so both channels are used. The `toMail()` and `toVonage()` methods format the messages. The `AnonymousNotifiable` is passed as the `$notifiable` argument, allowing the channel drivers to retrieve the routing information.

---

**Example 2: Sending a System Alert to an External Slack Webhook**

```php
<?php
// File: app/Notifications/SystemAlert.php

namespace App\Notifications;

use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\SlackMessage;

class SystemAlert extends Notification
{
    public function __construct(
        public string $title,
        public string $message,
        public string $severity,
    ) {}

    public function via(object $notifiable): array
    {
        return ['slack'];
    }

    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
            ->error()
            ->content('*' . $this->title . '*')
            ->attachment(function ($attachment) {
                $attachment->title('System Alert')
                    ->content($this->message)
                    ->fields([
                        'Severity' => $this->severity,
                        'Time'     => now()->toDateTimeString(),
                    ]);
            });
    }
}
```

```php
// Sending to a Slack webhook
use App\Notifications\SystemAlert;
use Illuminate\Support\Facades\Notification;

Notification::route('slack', 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX')
    ->notify(new SystemAlert(
        'Database Connection Lost',
        'The primary database connection has been lost. Failover initiated.',
        'critical'
    ));
```

**Expected Output:** A formatted Slack message is posted to the specified webhook channel with the alert title, message, and severity.

**Why This Output Occurs:** The `Notification::route('slack', $webhookUrl)` specifies the Slack webhook URL as the routing information. The `via()` method returns `['slack']`, so only the Slack channel is used. The `toSlack()` method builds a formatted `SlackMessage` with an error-level indicator, content, and attachment fields. The `AnonymousNotifiable` provides the webhook URL to the Slack channel driver.

### Real-World Cases

- **Guest checkout:** Sending order confirmations and shipping notifications to guest customers who provided an email address but did not create an account.
- **Vendor notifications:** Alerting external vendors about inventory shortages or delivery delays via email or Slack.
- **Emergency alerts:** Sending critical system alerts to on-call engineers via SMS and Slack without creating user accounts for each engineer.
- **Customer support escalations:** Notifying external stakeholders about support ticket escalations via email.
- **Marketing campaigns:** Sending promotional notifications to leads collected from landing pages.

---

## 4. Localization

### Definitions

**Core Definition:** Notification localization is the process of automatically translating notification content into the recipient's preferred language using the `locale()` method or the `HasLocalePreference` contract.

**Technical Definition:** The `Illuminate\Notifications\Notification` class provides a `locale()` method that sets the desired locale for a single notification. When the notification is sent, the `NotificationSender` stores the locale, wraps the formatting and delivery in `App::withLocale()`, and restores the previous locale afterward. For automatic localization, models can implement the `Illuminate\Contracts\Translation\HasLocalePreference` contract, which defines a `preferredLocale()` method returning the user's locale. The `NotificationSender::preferredLocale()` method checks for this contract and applies the returned locale automatically.

**Beginner-Friendly Explanation:** If your application serves users in multiple languages, you can send notifications in each user's preferred language. Laravel automatically switches the application's language to the user's locale when formatting the notification, then switches back. You can either specify the locale manually for a single notification or store it on the user model and let Laravel handle it automatically.

### Purposes

- To send notification content in the recipient's preferred language.
- To provide a multilingual user experience without manual translation logic.
- To automatically apply locale preferences stored on user models.
- To override the user's preferred locale for specific notifications when needed.
- To support translation files for notification content across multiple languages.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Manual locale for a single notification
$user->notify((new InvoicePaid($invoice))->locale('es'));

// Automatic locale via HasLocalePreference contract
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    public function preferredLocale(): string
    {
        return $this->locale; // e.g., 'de', 'fr', 'es'
    }
}

// Now notifications are automatically sent in the user's preferred locale
$user->notify(new OrderConfirmation($order));
```

**Component Breakdown:**

- `->locale('es')` — Sets the locale for a single notification instance.
- `HasLocalePreference` — An interface that models can implement to declare a preferred locale.
- `preferredLocale()` — The method that returns the user's locale string.
- The `NotificationSender` wraps delivery in `withLocale()` when a locale is set.

**Syntax Rules:**

- The `locale()` method must be called on the notification instance before sending.
- The `HasLocalePreference` contract must be implemented on the notifiable model (typically `User`).
- The `preferredLocale()` method must return a string locale code (e.g., `en`, `es`, `fr`, `de`).
- Translation files for notification content must exist in `lang/{locale}/` directories.
- The `locale()` method overrides the `HasLocalePreference` locale for that specific notification.

**Constraints and Limitations:**

- **The `HasLocalePreference` contract only applies to notifications and mailables sent to the model.** It does not affect other application translations.
- **The `locale()` method must be called before `notify()`.** Calling it after has no effect.
- **Translation files must exist for the target locale.** Without them, the fallback locale is used.
- **The `preferredLocale()` method must return a valid locale code.** Invalid codes may cause translation failures.
- **Database notifications store the translated content as-is.** The locale used at send time is not stored, so the content remains in the sent language.

### Annotated Code Examples

**Example 1: Manual Locale Override**

```php
<?php
// File: app/Notifications/InvoicePaid.php

namespace App\Notifications;

use App\Models\Invoice;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Invoice $invoice,
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject(__('Invoice Paid'))
            ->greeting(__('Hello :name!', ['name' => $notifiable->name]))
            ->line(__('Your invoice has been paid.'))
            ->action(__('View Invoice'), url('/invoices/' . $this->invoice->id));
    }
}
```

```php
// Sending in Spanish
$user->notify((new InvoicePaid($invoice))->locale('es'));

// Sending in German
$user->notify((new InvoicePaid($invoice))->locale('de'));

// Sending in the user's preferred locale (automatic)
$user->notify(new InvoicePaid($invoice));
```

**Expected Output:**

- The Spanish notification uses the translations from `lang/es/notifications.php` (or the notification's translation file).
- The German notification uses the translations from `lang/de/`.
- The automatic notification uses the locale returned by `preferredLocale()`.

**Why This Output Occurs:** The `locale('es')` call sets the locale on the notification instance. When the notification is sent, the `NotificationSender` calls `App::withLocale('es', fn () => $this->sendToNotifiable(...))`, which temporarily changes the application locale to Spanish. The `__()` helper then returns the Spanish translations. After delivery, the previous locale is restored.

---

**Example 2: Automatic Locale via HasLocalePreference**

```php
<?php
// File: app/Models/User.php

namespace App\Models;

use Illuminate\Contracts\Translation\HasLocalePreference;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements HasLocalePreference
{
    use Notifiable;

    protected $fillable = ['name', 'email', 'password', 'locale'];

    public function preferredLocale(): string
    {
        return $this->locale ?? config('app.locale');
    }
}
```

```php
// Creating a German user
$user = User::create([
    'name'     => 'Hans Müller',
    'email'    => 'hans@example.de',
    'password' => bcrypt('password'),
    'locale'   => 'de',
]);

// Sending a notification — automatically in German
$user->notify(new OrderConfirmation($order));

// Sending a notification — override to English
$user->notify((new OrderConfirmation($order))->locale('en'));
```

**Expected Output:**

- The first notification is sent in German using the `de` locale.
- The second notification is sent in English using the `en` locale, overriding the user's preference.

**Why This Output Occurs:** The `User` model implements `HasLocalePreference` and returns `'de'` from `preferredLocale()`. When the `NotificationSender` processes the notification, it checks for the `HasLocalePreference` contract on the notifiable, retrieves the preferred locale, and wraps delivery in `withLocale()`. The explicit `->locale('en')` call on the second notification overrides the user's preference for that specific send.

### Real-World Cases

- **International e-commerce:** Sending order confirmations, shipping updates, and invoices in the customer's local language.
- **Global SaaS platforms:** Notifying users about account activity in their preferred language.
- **Social networks:** Displaying notifications in the user's language across email, database, and broadcast channels.
- **Customer support:** Sending ticket updates in the language the customer used when submitting the ticket.
- **Travel booking:** Sending itinerary confirmations in the language of the booking.

---

## 5. Custom Channels

### Definitions

**Core Definition:** A custom notification channel is a user-defined driver that extends Laravel's notification system to deliver notifications through a channel not supported by the built-in drivers, such as a proprietary internal API, a custom webhook service, or an in-house messaging platform.

**Technical Definition:** A custom channel is a class that implements a `send()` method accepting two arguments: the `$notifiable` entity and the `$notification` instance. The channel class is typically placed in `app/Notifications/Channels/` and registered in the `via()` method of the notification by returning the fully qualified class name or a channel alias. The `ChannelManager` resolves the channel class from the container and calls its `send()` method. The notification class defines a message-building method (e.g., `toCustom()`) that returns the data structure the channel expects.

**Beginner-Friendly Explanation:** If Laravel doesn't have a built-in channel for the service you want to use — like a proprietary company messaging system or a custom webhook — you can create your own channel. You write a small class with a `send()` method that knows how to deliver the notification to that service. Then you add your channel to the notification's `via()` method, and Laravel handles the rest.

### Purposes

- To deliver notifications through services not supported by Laravel's built-in channels.
- To integrate with proprietary internal APIs and messaging platforms.
- To send notifications to custom webhook endpoints with bespoke payloads.
- To encapsulate channel-specific delivery logic in a reusable class.
- To extend the notification system without modifying Laravel's core.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Notifications/Channels/CustomWebhookChannel.php

namespace App\Notifications\Channels;

use Illuminate\Notifications\Notification;
use Illuminate\Support\Facades\Http;

class CustomWebhookChannel
{
    /**
     * Send the given notification.
     */
    public function send(object $notifiable, Notification $notification): void
    {
        // Step 1: Retrieve the message from the notification
        $message = $notification->toCustom($notifiable);

        // Step 2: Determine the webhook URL for the notifiable
        $url = $notifiable->routeNotificationFor('custom', $notification);

        // Step 3: Send the payload to the webhook
        Http::post($url, [
            'event'   => $message['event'],
            'payload' => $message['payload'],
            'sent_at' => now()->toIso8601String(),
        ]);
    }
}
```

```php
<?php
// File: app/Notifications/SystemAlert.php

namespace App\Notifications;

use App\Notifications\Channels\CustomWebhookChannel;
use Illuminate\Notifications\Notification;

class SystemAlert extends Notification
{
    public function __construct(
        public string $message,
        public string $severity,
    ) {}

    public function via(object $notifiable): array
    {
        return [CustomWebhookChannel::class];
    }

    public function toCustom(object $notifiable): array
    {
        return [
            'event'   => 'system.alert',
            'payload' => [
                'message'  => $this->message,
                'severity' => $this->severity,
            ],
        ];
    }
}
```

```php
// Sending the notification
$user->notify(new SystemAlert('Database connection lost.', 'critical'));
```

**Component Breakdown:**

- `send($notifiable, $notification)` — The required method signature for a custom channel.
- `$notification->toCustom($notifiable)` — Retrieves the message payload from the notification.
- `$notifiable->routeNotificationFor('custom', $notification)` — Retrieves the routing information for the custom channel.
- `via()` — Returns the fully qualified class name of the custom channel.

**Syntax Rules:**

- The channel class must have a `send()` method accepting `$notifiable` and `$notification` arguments.
- The notification class must define a message-building method (e.g., `toCustom()`) that returns the data the channel expects.
- The `via()` method returns the fully qualified class name of the channel or a registered alias.
- The channel class is resolved from the container, so constructor injection works.
- The `routeNotificationFor()` method on the notifiable provides the routing information for the custom channel.

**Constraints and Limitations:**

- **The `send()` method must handle its own error handling.** Unlike built-in channels, custom channels do not have automatic retry or failure handling.
- **The channel class must be registered or auto-discoverable.** If placed in a non-standard namespace, it must be resolvable by the container.
- **The `routeNotificationFor()` method must be defined on the notifiable.** Without it, the channel cannot determine the destination.
- **Custom channels do not support the `database` channel's automatic persistence.** You must implement any storage logic yourself.
- **The `send()` method is called synchronously unless the notification implements `ShouldQueue`.** For long-running HTTP calls, implement `ShouldQueue`.

### Annotated Code Examples

**Example 1: Custom Webhook Channel**

```php
<?php
// File: app/Notifications/Channels/WebhookChannel.php

namespace App\Notifications\Channels;

use Illuminate\Notifications\Notification;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class WebhookChannel
{
    public function send(object $notifiable, Notification $notification): void
    {
        // Step 1: Get the webhook URL from the notifiable
        $url = $notifiable->routeNotificationFor('webhook', $notification);

        if (!$url) {
            Log::warning('Webhook channel: no URL configured for notifiable.', [
                'notifiable_id' => $notifiable->id ?? 'anonymous',
            ]);
            return;
        }

        // Step 2: Get the payload from the notification
        $payload = $notification->toWebhook($notifiable);

        // Step 3: Send the HTTP POST request
        $response = Http::timeout(10)
            ->withHeaders([
                'X-Notification-Id' => (string) \Str::uuid(),
                'Content-Type'      => 'application/json',
            ])
            ->post($url, $payload);

        // Step 4: Log the result for auditing
        if ($response->successful()) {
            Log::info('Webhook notification delivered.', [
                'url'    => $url,
                'status' => $response->status(),
            ]);
        } else {
            Log::error('Webhook notification failed.', [
                'url'    => $url,
                'status' => $response->status(),
                'body'   => $response->body(),
            ]);
        }
    }
}
```

```php
<?php
// File: app/Notifications/OrderStatusChanged.php

namespace App\Notifications;

use App\Models\Order;
use App\Notifications\Channels\WebhookChannel;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class OrderStatusChanged extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order,
        public string $oldStatus,
        public string $newStatus,
    ) {}

    public function via(object $notifiable): array
    {
        return [WebhookChannel::class];
    }

    public function toWebhook(object $notifiable): array
    {
        return [
            'event'       => 'order.status_changed',
            'order_id'    => $this->order->id,
            'order_number' => $this->order->number,
            'old_status'  => $this->oldStatus,
            'new_status'  => $this->newStatus,
            'changed_at'  => now()->toIso8601String(),
        ];
    }
}
```

```php
// Sending the notification
$order = Order::find(123);
$order->customer->notify(new OrderStatusChanged($order, 'processing', 'shipped'));
```

**Expected Output:** A POST request is sent to the customer's configured webhook URL with the order status change payload. The result is logged for auditing.

**Why This Output Occurs:** The `via()` method returns the `WebhookChannel` class name. The `ChannelManager` resolves the channel from the container and calls its `send()` method. The channel retrieves the webhook URL via `routeNotificationFor('webhook', $notification)`, gets the payload from `toWebhook()`, and sends the HTTP POST request. The `ShouldQueue` interface ensures the notification is queued, so the HTTP call does not block the request.

---

**Example 2: Custom Internal API Channel**

```php
<?php
// File: app/Notifications/Channels/InternalApiChannel.php

namespace App\Notifications\Channels;

use Illuminate\Notifications\Notification;
use Illuminate\Support\Facades\Http;

class InternalApiChannel
{
    public function __construct(
        private string $apiKey,
        private string $baseUrl,
    ) {}

    public function send(object $notifiable, Notification $notification): void
    {
        $message = $notification->toInternalApi($notifiable);

        Http::withToken($this->apiKey)
            ->timeout(15)
            ->post($this->baseUrl . '/notifications', [
                'recipient_id' => $notifiable->id,
                'type'         => $message['type'],
                'title'        => $message['title'],
                'body'         => $message['body'],
                'metadata'     => $message['metadata'] ?? [],
            ]);
    }
}
```

```php
<?php
// File: app/Providers/NotificationServiceProvider.php

namespace App\Providers;

use App\Notifications\Channels\InternalApiChannel;
use Illuminate\Support\ServiceProvider;

class NotificationServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(InternalApiChannel::class, function ($app) {
            return new InternalApiChannel(
                apiKey: config('services.internal_api.key'),
                baseUrl: config('services.internal_api.url'),
            );
        });
    }
}
```

```php
<?php
// File: app/Notifications/AccountActivity.php

namespace App\Notifications;

use App\Notifications\Channels\InternalApiChannel;
use Illuminate\Notifications\Notification;

class AccountActivity extends Notification
{
    public function __construct(
        public string $activityType,
        public string $description,
    ) {}

    public function via(object $notifiable): array
    {
        return [InternalApiChannel::class];
    }

    public function toInternalApi(object $notifiable): array
    {
        return [
            'type'     => 'account.activity',
            'title'    => 'Account Activity',
            'body'     => $this->description,
            'metadata' => [
                'activity_type' => $this->activityType,
                'ip_address'    => request()->ip(),
            ],
        ];
    }
}
```

**Expected Output:** A POST request is sent to the internal API with the account activity data. The API processes the notification and delivers it through the company's internal messaging platform.

**Why This Output Occurs:** The `InternalApiChannel` is resolved from the container with its dependencies injected (API key and base URL from configuration). The channel's `send()` method retrieves the payload from `toInternalApi()` and sends it to the internal API. The channel is bound in the service provider, allowing configuration values to be injected without hardcoding.

### Real-World Cases

- **Proprietary messaging platforms:** Sending notifications to an internal company messaging system (e.g., a custom-built Slack alternative) via a REST API.
- **Custom webhooks:** Delivering notifications to external partner systems via HTTP webhooks with bespoke payloads.
- **SMS gateways:** Integrating with a regional SMS provider that is not supported by Laravel's built-in Vonage channel.
- **Push notification services:** Sending push notifications through Firebase Cloud Messaging (FCM) or Apple Push Notification Service (APNs) via a custom channel.
- **IoT alerting:** Sending notifications to IoT devices or MQTT brokers via a custom channel that publishes to a message queue.

---

## References

- Laravel Notifications Documentation (Master) — https://laravel.com/framework/docs/master/notifications
- Laravel Notifications Documentation (Laravel 13.x) — https://laravel.com/docs/13.x/notifications
- Laravel Notifications Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/notifications
- Laravel Notifications Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/notifications
- Laravel `Illuminate\Notifications\Notification` API — https://api.laravel.com/docs/master/Illuminate/Notifications/Notification.html
- Laravel `Illuminate\Notifications\NotificationSender` API — https://api.laravel.com/docs/master/Illuminate/Notifications/NotificationSender.html
- Laravel `Illuminate\Notifications\ChannelManager` API — https://api.laravel.com/docs/master/Illuminate/Notifications/ChannelManager.html
- Laravel `Illuminate\Contracts\Translation\HasLocalePreference` API — https://api.laravel.com/docs/master/Illuminate/Contracts/Translation/HasLocalePreference.html
- Laravel `Illuminate\Notifications\AnonymousNotifiable` API — https://api.laravel.com/docs/master/Illuminate/Notifications/AnonymousNotifiable.html
- Laravel `Illuminate\Notifications\Messages\MailMessage` API — https://api.laravel.com/docs/master/Illuminate/Notifications/Messages/MailMessage.html
- Laravel `Illuminate\Notifications\Messages\BroadcastMessage` API — https://api.laravel.com/docs/master/Illuminate/Notifications/Messages/BroadcastMessage.html
- Laravel `Illuminate\Notifications\Messages\VonageMessage` API — https://api.laravel.com/docs/master/Illuminate/Notifications/Messages/VonageMessage.html
- Laravel `Illuminate\Notifications\Messages\SlackMessage` API — https://api.laravel.com/docs/master/Illuminate/Notifications/Messages/SlackMessage.html
- Laravel Notification Channels Community — https://laravel-notification-channels.com/
- Laravel 5.7 Mail Localization Improvements (Laravel News) — https://laravel-news.com/mail-localization
- Using User Locale for Notifications (Dev.to) — https://dev.to/using-user-locale-for-notifications
- Building Cross-Platform Alerts with Laravel's Notification Framework (Laravel News) — https://laravel-news.com/building-cross-platform-alerts-with-laravels-notification-framework
- Send Notifications to Anyone (Laravel Daily) — https://laraveldaily.com/send-notifications-to-anyone
- Laravel Notification Channels, Under the Hood — https://laravel-notification-channels.com/about/
- Laravel Reverb Documentation — https://laravel.com/docs/master/reverb
- Laravel Broadcasting Documentation — https://laravel.com/docs/master/broadcasting