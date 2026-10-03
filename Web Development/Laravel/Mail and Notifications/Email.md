# Laravel Email & Outbound Messaging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Laravel Email & Outbound Messaging is the framework's subsystem for composing, configuring, and dispatching email messages through multiple transport drivers, using mailable classes, Markdown templates, and dynamic attachment handling, with built-in testing and previewing capabilities.

**Technical Definition:** Laravel's mail system is built on the Symfony Mailer component, wrapped by `Illuminate\Mail\MailManager` and exposed through the `Illuminate\Support\Facades\Mail` facade. Mail is configured in `config/mail.php` via a `mailers` array, where each mailer defines a transport (`smtp`, `mailgun`, `postmark`, `ses`, `log`, etc.) and its credentials. Messages are represented as mailable classes extending `Illuminate\Mail\Mailable`, which define their interface through `envelope()`, `content()`, and `attachments()` methods returning `Envelope`, `Content`, and `Attachment` objects respectively. Markdown mailables use Blade components and Markdown syntax to render responsive HTML emails. Testing is provided by `Illuminate\Support\Testing\Fakes\MailFake`, accessible via `Mail::fake()`, which records sent and queued mailables for assertion.

**Beginner-Friendly Explanation:** Laravel makes sending emails simple. You create a "mailable" class that describes the email — who it's from, what the subject is, which template to use, and what files to attach. You configure how emails are sent (through Gmail, Mailgun, Amazon SES, or just logged to a file for testing). Laravel handles all the technical details, whether you're sending a simple text email or a beautifully styled Markdown template with attachments. For testing, you can pretend to send emails and check that the right emails were sent to the right people.

### Key Characteristics

- **Multi-transport support:** SMTP, Mailgun, Postmark, Amazon SES, Resend, MailerSend, Sendmail, Log, Array, and more.
- **Mailable classes:** Each email type is a dedicated PHP class with `envelope()`, `content()`, and `attachments()` methods.
- **Markdown mail:** Responsive email templates built with Blade components and Markdown syntax, pre-styled with Tailwind CSS.
- **Dynamic attachments:** Attach files from local paths, storage disks, raw data strings, or attachable model objects.
- **Failover and load balancing:** The `failover` transport provides high availability; the `roundrobin` transport distributes load across multiple mailers.
- **Testing support:** `Mail::fake()` records sent and queued mailables; assertions verify recipients, subject, content, and attachments.
- **Local previewing:** The `log` driver writes emails to `storage/logs/laravel.log`; Mailpit and MailHog provide web-based preview interfaces.

### Prerequisites

- PHP 8.1 or higher (Laravel 10+; Laravel 11 requires PHP 8.2+).
- A Laravel application with `config/mail.php` present.
- For API-based drivers (Mailgun, Postmark, SES): the corresponding Symfony Mailer transport package installed via Composer.
- For Markdown mail: no additional dependencies (built into Laravel).
- For testing: PHPUnit or Pest installed and configured.

### Related Programming Areas

- **Symfony Mailer** — The underlying component providing transport drivers and message composition.
- **Blade Templating** — Used for both HTML and Markdown email templates.
- **Queue System** — Mailables can implement `ShouldQueue` for background sending.
- **Notifications** — Mail notifications build on the mail system.
- **Storage** — Attachments can be pulled from any configured filesystem disk.
- **Testing** — Laravel's testing framework provides `Mail::fake()` and assertion helpers.

### Core Concepts / Features

1. Mail Configuration (Setting up multi-transport drivers: SMTP, Mailgun, Postmark, SES, Log)
2. Mailable Classes (Building modern envelope and content architectures: Envelope, Content, Attachment)
3. Markdown Mail (Styling responsive layouts with integrated Tailwind/Blade syntax components)
4. Dynamic Attachments (Attaching raw data string formats, cloud storage files, and dynamic naming parameters)
5. Testing Mailables (Leveraging modern testing assertions and visual previewing in local logs)

---

## 1. Mail Configuration

### Definitions

**Core Definition:** Mail configuration is the process of defining mail transport drivers, credentials, and default sender information in `config/mail.php` and `.env`, enabling the application to send email through one or more delivery services.

**Technical Definition:** The `config/mail.php` file contains a `default` key (specifying the default mailer name) and a `mailers` array where each key is a mailer name and each value is a configuration array with a `transport` key and driver-specific options. Laravel's `MailManager` reads this configuration and resolves the appropriate Symfony Mailer transport via the `TransportManager`. Supported transports include `smtp`, `sendmail`, `mailgun`, `ses`, `ses-v2`, `postmark`, `resend`, `mailersend`, `log`, `array`, `failover`, and `roundrobin`. Credentials are injected via environment variables, typically stored in `.env`.

**Beginner-Friendly Explanation:** Mail configuration is where you tell Laravel how to send emails. You might use Gmail's SMTP server for development, Mailgun for production, and just log emails to a file when testing. Laravel lets you define multiple "mailers" — one for each service — and choose which one to use by default. You put your API keys and passwords in the `.env` file, which is never committed to version control.

### Purposes

- To define one or more mail transport drivers with their credentials and options.
- To select a default mailer for the application.
- To configure a global "from" address and name for all outgoing emails.
- To set up failover or round-robin configurations for high availability and load balancing.
- To enable local development email previewing through the `log` or `array` drivers.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// config/mail.php

return [
    // Default mailer
    'default' => env('MAIL_MAILER', 'log'),

    'mailers' => [
        'smtp' => [
            'transport' => 'smtp',
            'host'      => env('MAIL_HOST', '127.0.0.1'),
            'port'      => env('MAIL_PORT', 2525),
            'username'  => env('MAIL_USERNAME'),
            'password'  => env('MAIL_PASSWORD'),
            'encryption' => env('MAIL_ENCRYPTION', 'tls'),
            'timeout'   => null,
        ],

        'mailgun' => [
            'transport' => 'mailgun',
            'domain'    => env('MAILGUN_DOMAIN'),
            'secret'    => env('MAILGUN_SECRET'),
            'endpoint'  => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
        ],

        'postmark' => [
            'transport' => 'postmark',
            'token'     => env('POSTMARK_TOKEN'),
            'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
        ],

        'ses' => [
            'transport' => 'ses',
            'key'       => env('AWS_ACCESS_KEY_ID'),
            'secret'    => env('AWS_SECRET_ACCESS_KEY'),
            'region'    => env('AWS_DEFAULT_REGION', 'us-east-1'),
        ],

        'log' => [
            'transport' => 'log',
            'channel'   => env('MAIL_LOG_CHANNEL'),
        ],

        'array' => [
            'transport' => 'array',
        ],

        'failover' => [
            'transport' => 'failover',
            'mailers'   => ['postmark', 'mailgun', 'sendmail'],
        ],

        'roundrobin' => [
            'transport' => 'roundrobin',
            'mailers'   => ['ses', 'postmark'],
        ],
    ],

    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
        'name'    => env('MAIL_FROM_NAME', 'Example'),
    ],

    'reply_to' => [
        'address' => env('MAIL_REPLY_TO_ADDRESS', 'support@example.com'),
        'name'    => env('MAIL_REPLY_TO_NAME', 'Support'),
    ],
];
```

**Component Breakdown:**

- `'default'` — The name of the mailer used when no explicit mailer is specified. Controlled by `MAIL_MAILER`.
- `'mailers'` — Associative array where each key is a mailer name and each value is a configuration array.
- `'transport'` — The driver name (`smtp`, `mailgun`, `postmark`, `ses`, `log`, `array`, `failover`, `roundrobin`).
- `'from'` — Global sender address and name applied to all mailables unless overridden.
- `'reply_to'` — Global reply-to address and name.

**Syntax Rules:**

- The `default` key must match one of the keys in the `mailers` array.
- API-based drivers (`mailgun`, `postmark`, `ses`) require their Symfony Mailer transport packages to be installed via Composer.
- The `failover` transport tries each configured mailer in order until one succeeds.
- The `roundrobin` transport cycles through the configured mailers, distributing load evenly.
- The `log` driver writes the full email (headers and body) to the configured log channel or `storage/logs/laravel.log`.

**Constraints and Limitations:**

- **API-based drivers require additional Composer packages.** For example, Mailgun requires `symfony/mailgun-mailer` and `symfony/http-client`.
- **The `log` driver does not send emails.** It only writes them to the log file for inspection.
- **The `array` driver stores emails in memory only.** They are discarded when the request ends; use it for testing.
- **Failover and roundrobin transports do not work with the `log` or `array` drivers** as primary options in production scenarios.
- **Credentials in `config/mail.php` should always come from environment variables.** Never hardcode secrets.

### Annotated Code Examples

**Example 1: Configuring Multiple Mailers**

```php
// .env file
MAIL_MAILER=smtp

MAIL_HOST=smtp.mailgun.org
MAIL_PORT=587
MAIL_USERNAME=postmaster@mg.example.com
MAIL_PASSWORD=your-mailgun-smtp-password
MAIL_ENCRYPTION=tls

MAILGUN_DOMAIN=mg.example.com
MAILGUN_SECRET=your-mailgun-api-key

POSTMARK_TOKEN=your-postmark-token

AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
AWS_DEFAULT_REGION=us-east-1

MAIL_FROM_ADDRESS=hello@example.com
MAIL_FROM_NAME="Example App"
```

```php
// config/mail.php
'default' => env('MAIL_MAILER', 'smtp'),

'mailers' => [
    'smtp' => [
        'transport' => 'smtp',
        'host'      => env('MAIL_HOST', 'smtp.mailgun.org'),
        'port'      => env('MAIL_PORT', 587),
        'username'  => env('MAIL_USERNAME'),
        'password'  => env('MAIL_PASSWORD'),
        'encryption' => env('MAIL_ENCRYPTION', 'tls'),
    ],

    'postmark' => [
        'transport' => 'postmark',
        'token'     => env('POSTMARK_TOKEN'),
    ],

    'ses' => [
        'transport' => 'ses',
        'key'       => env('AWS_ACCESS_KEY_ID'),
        'secret'    => env('AWS_SECRET_ACCESS_KEY'),
        'region'    => env('AWS_DEFAULT_REGION'),
    ],

    'failover' => [
        'transport' => 'failover',
        'mailers'   => ['postmark', 'ses', 'smtp'],
    ],
],
```

```php
// Usage — send via the default (smtp) mailer
Mail::to('user@example.com')->send(new OrderShipped($order));

// Usage — send via a specific mailer
Mail::mailer('postmark')->to('user@example.com')->send(new OrderShipped($order));

// Usage — send via the failover mailer
Mail::mailer('failover')->to('user@example.com')->send(new OrderShipped($order));
```

**Expected Output:**

- The email is sent via the configured SMTP server (Mailgun SMTP relay).
- If the `postmark` mailer is used, the email is sent via the Postmark API.
- If the `failover` mailer is used, Laravel tries Postmark first, then SES, then SMTP.

**Why This Output Occurs:** The `default` key selects the `smtp` mailer. The `failover` mailer tries each configured mailer in order until one succeeds. Each mailer's transport is resolved by the `TransportManager` based on the `transport` key, using the credentials from the configuration array.

---

**Example 2: Local Development with the Log Driver**

```php
// .env (local development)
MAIL_MAILER=log
```

```php
// config/mail.php
'default' => env('MAIL_MAILER', 'log'),

'mailers' => [
    'log' => [
        'transport' => 'log',
        'channel'   => env('MAIL_LOG_CHANNEL'),
    ],
],
```

```php
// Sending a test email
Mail::raw('This is a test email.', function ($message) {
    $message->to('dev@example.com')
            ->subject('Test Email');
});
```

**Expected Output (in `storage/logs/laravel.log`):**

```
[2025-06-15 10:00:00] local.INFO: Message-ID: <abc123@example.com>
Date: Sun, 15 Jun 2025 10:00:00 +0000
From: Example <hello@example.com>
To: dev@example.com
Subject: Test Email
MIME-Version: 1.0
Content-Type: text/plain; charset=utf-8

This is a test email.
```

**Why This Output Occurs:** The `log` driver writes the full email — headers and body — to the configured log channel or the default `laravel.log` file. This allows developers to inspect email content without sending it to a real recipient.

### Real-World Cases

- **Transactional emails:** Use Postmark or SES for password resets, order confirmations, and receipts.
- **Bulk newsletters:** Use Mailgun or Sendmail for high-volume marketing emails.
- **Local development:** Use the `log` driver to inspect email content without sending real emails.
- **High availability:** Use the `failover` transport with Postmark and SES to ensure emails are sent even if one provider is down.
- **Load balancing:** Use the `roundrobin` transport with multiple providers to distribute sending volume and avoid rate limits.

---

## 2. Mailable Classes

### Definitions

**Core Definition:** A mailable class is a PHP class extending `Illuminate\Mail\Mailable` that represents a single email type, defining its envelope (sender, subject, recipients), content (Blade template and data), and attachments.

**Technical Definition:** The `Mailable` class provides three primary methods: `envelope()` returns an `Illuminate\Mail\Mailables\Envelope` object defining the subject, from, reply-to, cc, bcc, and tags; `content()` returns an `Illuminate\Mail\Mailables\Content` object defining the Blade view, plain-text view, Markdown view, and view data; `attachments()` returns an array of `Illuminate\Mail\Mailables\Attachment` objects. The `build()` method (Laravel 8 and below) is deprecated in favour of the envelope/content architecture (Laravel 9+). Mailables can implement `ShouldQueue` to be queued for background sending.

**Beginner-Friendly Explanation:** A mailable class is a PHP class that describes one type of email your application sends. You create one for each email — an "order shipped" email, a "welcome" email, a "password reset" email. The class defines who the email is from, what the subject is, which template to use, and what files to attach. Then you send it with `Mail::to($user)->send(new OrderShipped($order))`.

### Purposes

- To encapsulate the definition of a single email type in a dedicated, testable class.
- To separate email content (Blade templates) from email logic (mailable class).
- To enable dependency injection of models and services into email content.
- To support queuing for background sending via the `ShouldQueue` interface.
- To provide a consistent, object-oriented interface for composing emails.

### Syntax Rules and Structure

#### Complete General Syntax

```php
<?php
// File: app/Mail/OrderShipped.php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Mail\Mailables\Attachment;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            from: new Address('billing@example.com', 'Billing Team'),
            replyTo: [new Address('support@example.com', 'Support')],
            subject: 'Order Shipped',
            tags: ['order', 'shipping'],
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            text: 'mail.orders.shipped-text',
            with: [
                'orderNumber' => $this->order->number,
                'trackingUrl' => $this->order->tracking_url,
            ],
        );
    }

    public function attachments(): array
    {
        return [
            Attachment::fromPath(storage_path('app/invoices/' . $this->order->id . '.pdf'))
                ->as('invoice.pdf')
                ->withMime('application/pdf'),
        ];
    }
}
```

**Component Breakdown:**

- `__construct()` — Receives the data needed to build the email (models, strings, etc.). Public properties are automatically available in the view.
- `envelope()` — Returns an `Envelope` object defining the sender, recipients, subject, and tags.
- `content()` — Returns a `Content` object defining the HTML view, plain-text view, Markdown view, and view data.
- `attachments()` — Returns an array of `Attachment` objects.

**Syntax Rules:**

- The `envelope()` method must return an `Envelope` instance. In Laravel 11+, this is required; the old `build()` method is deprecated.
- The `content()` method must return a `Content` instance.
- Public properties on the mailable class are automatically available in the view.
- Use the `with` parameter in `Content` to pass data to the view when properties are protected or private.
- Mailables can implement `ShouldQueue` to be queued automatically when sent.

**Constraints and Limitations:**

- **The `build()` method is deprecated.** Use `envelope()` and `content()` instead.
- **The `from` address in the envelope overrides the global `from` in `config/mail.php`.** Use the global configuration to avoid repetition.
- **Attachments must exist at send time.** If an attachment file is missing, the email will fail to send.
- **Serialised models in queued mailables may become stale.** Use `SerializesModels` and consider fresh model loading.

### Annotated Code Examples

**Example 1: Complete Mailable with Envelope, Content, and Attachments**

```php
<?php
// File: app/Mail/OrderShipped.php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Attachment;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            from: new Address('billing@example.com', 'Billing Team'),
            replyTo: [
                new Address('support@example.com', 'Support'),
            ],
            subject: 'Order #' . $this->order->number . ' Shipped',
            tags: ['order', 'shipping'],
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            text: 'mail.orders.shipped-text',
            with: [
                'orderNumber' => $this->order->number,
                'customerName' => $this->order->customer->name,
                'trackingUrl'  => $this->order->tracking_url,
            ],
        );
    }

    public function attachments(): array
    {
        return [
            Attachment::fromPath(
                storage_path('app/invoices/' . $this->order->id . '.pdf')
            )
                ->as('invoice-' . $this->order->number . '.pdf')
                ->withMime('application/pdf'),
        ];
    }
}
```

```php
// Sending the mailable
use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

$order = Order::find(123);
Mail::to($order->customer->email)->send(new OrderShipped($order));
```

**Expected Output:** The email is sent to the customer with the subject "Order #12345 Shipped", the HTML body rendered from `mail.orders.shipped`, and the invoice PDF attached.

**Why This Output Occurs:** The `envelope()` method defines the sender, reply-to, subject, and tags. The `content()` method defines the HTML view, plain-text view, and view data. The `attachments()` method attaches the invoice PDF with a dynamic filename. When `Mail::to()->send()` is called, Laravel resolves the mailable, renders the view with the provided data, attaches the file, and sends the message via the default mailer.

---

**Example 2: Plain-Text and HTML Versions**

```php
<?php
// File: app/Mail/InvoicePaid.php

namespace App\Mail;

use App\Models\Invoice;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class InvoicePaid extends Mailable
{
    public function __construct(
        public Invoice $invoice,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Invoice #' . $this->invoice->number . ' Paid',
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.invoices.paid',
            text: 'mail.invoices.paid-text',
            with: [
                'invoiceNumber' => $this->invoice->number,
                'amount'        => $this->invoice->formatted_amount,
                'paidAt'        => $this->invoice->paid_at->format('F j, Y'),
            ],
        );
    }
}
```

```blade
{{-- resources/views/mail/invoices/paid.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>Invoice Paid</title>
</head>
<body>
    <h1>Invoice Paid</h1>
    <p>Dear {{ $invoice->customer->name }},</p>
    <p>Thank you for your payment of {{ $amount }} for invoice #{{ $invoiceNumber }}.</p>
    <p>Payment received on {{ $paidAt }}.</p>
</body>
</html>
```

```blade
{{-- resources/views/mail/invoices/paid-text.blade.php --}}
Invoice Paid

Dear {{ $invoice->customer->name }},

Thank you for your payment of {{ $amount }} for invoice #{{ $invoiceNumber }}.
Payment received on {{ $paidAt }}.
```

**Expected Output:** The email client displays the HTML version if supported; otherwise, the plain-text version is displayed.

**Why This Output Occurs:** The `content()` method specifies both `view` (HTML) and `text` (plain-text) templates. Laravel sends both versions in a multipart email. Email clients that support HTML render the HTML version; text-only clients display the plain-text version.

### Real-World Cases

- **Order confirmations:** A mailable class that includes the order details, shipping address, and a PDF invoice attachment.
- **Password reset emails:** A mailable class that includes a time-limited reset link and security instructions.
- **Welcome emails:** A mailable class that greets new users and includes a link to complete their profile.
- **Invoice notifications:** A mailable class that includes the invoice PDF and payment instructions.
- **Weekly digests:** A mailable class that summarises user activity and includes a link to view more details.

---

## 3. Markdown Mail

### Definitions

**Core Definition:** Markdown mail is Laravel's system for building responsive HTML email templates using a combination of Markdown syntax and pre-built Blade components, styled with a Tailwind CSS-based theme.

**Technical Definition:** Markdown mailables are defined by setting the `markdown` key in the `Content` object returned by the `content()` method. The Markdown renderer (`Illuminate\Mail\Markdown`) parses the Markdown template into HTML, injecting Blade components such as `mail::message`, `mail::button`, `mail::panel`, and `mail::table`. The default theme is defined in `resources/views/vendor/mail/html/themes/default.css`, which can be customised by publishing the mail views and modifying the CSS. The `theme` property on the mailable class or the `markdown.theme` configuration key selects the active theme.

**Beginner-Friendly Explanation:** Markdown mail lets you write emails using simple Markdown syntax (like `# Heading` and `**bold**`) while still getting a beautiful, responsive HTML email. Laravel provides pre-built components — buttons, panels, tables — that you can use in your Markdown templates. The result looks professional on any device, and you don't have to write complex HTML.

### Purposes

- To build responsive HTML emails with minimal effort using Markdown syntax.
- To leverage pre-built Blade components for common email elements (buttons, panels, tables).
- To provide a consistent, branded look across all application emails.
- To allow customisation of the email theme through CSS and component overrides.
- To simplify the creation of plain-text alternatives alongside HTML versions.

### Syntax Rules and Structure

#### Complete General Syntax

```php
// Generating a Markdown mailable
php artisan make:mail OrderShipped --markdown=mail.orders.shipped

// Mailable content method with Markdown
public function content(): Content
{
    return new Content(
        markdown: 'mail.orders.shipped',
        with: [
            'orderNumber' => $this->order->number,
        ],
    );
}
```

```blade
{{-- resources/views/mail/orders/shipped.blade.php --}}
@component('mail::message')
# Order Shipped

Your order #{{ $orderNumber }} has been shipped!

@component('mail::button', ['url' => $trackingUrl])
Track Your Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

```blade
{{-- resources/views/mail/orders/shipped.blade.php (with panel and table) --}}
@component('mail::message')
# Order Summary

Your order has been shipped.

@component('mail::panel')
This is the panel content. You can put important notices here.
@endcomponent

@component('mail::table')
| Item       | Price   |
| ---------- | -------:|
| Product A  | $10.00  |
| Product B  | $20.00  |
@endcomponent

@component('mail::button', ['url' => $url, 'color' => 'success'])
View Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

**Component Breakdown:**

- `@component('mail::message')` — The wrapper component for all Markdown emails.
- `@component('mail::button', ['url' => '...', 'color' => '...'])` — Renders a centred button link. Supported colours: `primary`, `success`, `error`.
- `@component('mail::panel')` — Renders a block of text in a panel with a slightly different background colour.
- `@component('mail::table')` — Converts a Markdown table into an HTML table.
- `@component('mail::message')` — Must wrap all content.

**Syntax Rules:**

- Markdown templates use the `@component` directive with the `mail::` namespace.
- Standard Markdown syntax (`#`, `**`, `_`, lists, links) is supported within the components.
- The `markdown` key in `Content` specifies the Markdown template path.
- The `theme` property on the mailable class or the `markdown.theme` configuration key selects the active theme.
- A plain-text version is automatically generated from the Markdown template unless explicitly overridden.

**Constraints and Limitations:**

- **Markdown mail requires Blade components.** You cannot use pure Markdown without the `mail::message` wrapper.
- **The default theme uses Tailwind CSS.** Custom themes require publishing the mail views and modifying the CSS.
- **Complex layouts may require custom Blade components.** The pre-built components cover common cases but not every design.
- **Inline images are supported via `$message->embed()`** in Markdown templates, but raw data embedding requires the `$message->embedData()` method.

### Annotated Code Examples

**Example 1: Generating and Using a Markdown Mailable**

```bash
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

**Expected Output:**

```
INFO  Mailable [app/Mail/OrderShipped.php] created successfully.
INFO  Markdown view [resources/views/mail/orders/shipped.blade.php] created successfully.
```

**Generated Mailable:**

```php
<?php
// File: app/Mail/OrderShipped.php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    public function __construct(
        public Order $order,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Order Shipped',
        );
    }

    public function content(): Content
    {
        return new Content(
            markdown: 'mail.orders.shipped',
            with: [
                'orderNumber' => $this->order->number,
                'trackingUrl' => $this->order->tracking_url,
            ],
        );
    }

    public function attachments(): array
    {
        return [];
    }
}
```

**Generated Markdown View:**

```blade
{{-- resources/views/mail/orders/shipped.blade.php --}}
@component('mail::message')
# Order Shipped

Your order #{{ $orderNumber }} has been shipped!

@component('mail::button', ['url' => $trackingUrl])
Track Your Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

```php
// Sending the Markdown mailable
Mail::to($order->customer->email)->send(new OrderShipped($order));
```

**Expected Output:** The customer receives a beautifully styled HTML email with a centred button, rendered from the Markdown template.

**Why This Output Occurs:** The `markdown` key in `Content` tells Laravel to render the specified template using the Markdown renderer. The `@component('mail::message')` wrapper provides the email layout (header, footer, responsive styles). The `@component('mail::button')` renders a styled button with the provided URL. Laravel automatically generates a plain-text version from the Markdown template.

---

**Example 2: Customising the Markdown Theme**

```bash
# Publish the mail views
php artisan vendor:publish --tag=laravel-mail
```

**Expected Output:**

```
Copied Directory [vendor/laravel/framework/src/Illuminate/Mail/resources/views] to [resources/views/vendor/mail]
```

**Published Files:**

- `resources/views/vendor/mail/html/themes/default.css` — The default theme CSS.
- `resources/views/vendor/mail/html/` — The HTML components.
- `resources/views/vendor/mail/text/` — The plain-text components.

```css
/* resources/views/vendor/mail/html/themes/default.css */
/* Customise the theme colours */
.button-primary {
    background-color: #4f46e5;
    border-top: 10px solid #4f46e5;
    border-right: 18px solid #4f46e5;
    border-bottom: 10px solid #4f46e5;
    border-left: 18px solid #4f46e5;
}
```

```php
// config/mail.php
'markdown' => [
    'theme' => 'default',
    'paths' => [
        resource_path('views/vendor/mail'),
    ],
],
```

**Expected Output:** All Markdown emails now use the customised button colour (`#4f46e5` instead of the default).

**Why This Output Occurs:** Publishing the mail views copies Laravel's default Markdown components and CSS to `resources/views/vendor/mail`. Modifying `default.css` overrides the default theme. The `markdown.theme` configuration key tells Laravel to use the `default` theme from the published paths.

### Real-World Cases

- **Transactional emails:** Order confirmations, shipping notifications, and receipts styled with the Markdown components.
- **Password reset emails:** A Markdown mailable with a prominent reset button.
- **Welcome emails:** A Markdown mailable with a panel highlighting key features and a button linking to the dashboard.
- **Invoice notifications:** A Markdown mailable with a table listing line items and a button to view the full invoice.
- **Branded emails:** Custom themes that match the application's brand colours and typography.

---

## 4. Dynamic Attachments

### Definitions

**Core Definition:** Dynamic attachments are files or data attached to an email at runtime, sourced from local paths, storage disks, raw data strings, or attachable model objects, with optional custom filenames and MIME types.

**Technical Definition:** The `Illuminate\Mail\Mailables\Attachment` class provides static factory methods for creating attachments: `fromPath($path)` for local files, `fromStorage($path)` for files on the default storage disk, `fromStorageDisk($disk, $path)` for files on a specific disk, `fromData(Closure $data, $name)` for raw data strings, and `fromUrl($url)` for remote URLs. The `as($name)` method sets the display filename, and `withMime($mime)` sets the MIME type. The `Attachable` interface (`Illuminate\Contracts\Mail\Attachable`) allows model objects to define a `toMailAttachment()` method that returns an `Attachment` instance.

**Beginner-Friendly Explanation:** Sometimes you need to attach files to emails that aren't known ahead of time — like a PDF generated on the fly, a file stored on Amazon S3, or a model's associated document. Laravel lets you attach files from anywhere: local paths, cloud storage, raw data strings, or model objects. You can also rename the attachment and specify its MIME type.

### Purposes

- To attach files from local paths, cloud storage, or raw data strings.
- To attach files with dynamic filenames based on model data.
- To attach model objects directly via the `Attachable` interface.
- To specify the MIME type of attached files for proper handling.
- To embed inline images in email templates.

### Syntax Rules and Structure

#### Complete General Syntax

```php
use Illuminate\Mail\Mailables\Attachment;

// From a local path
Attachment::fromPath('/path/to/file')
    ->as('custom-name.pdf')
    ->withMime('application/pdf');

// From the default storage disk
Attachment::fromStorage('/path/to/file')
    ->as('name.pdf')
    ->withMime('application/pdf');

// From a specific storage disk
Attachment::fromStorageDisk('s3', '/path/to/file')
    ->as('name.pdf')
    ->withMime('application/pdf');

// From raw data
Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
    ->withMime('application/pdf');

// From a URL
Attachment::fromUrl('https://example.com/file.pdf')
    ->as('downloaded.pdf');

// From an attachable model
class Photo extends Model implements Attachable
{
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath($this->path)
            ->as($this->name)
            ->withMime($this->mime_type);
    }
}
```

**Component Breakdown:**

- `fromPath($path)` — Creates an attachment from a local file path.
- `fromStorage($path)` — Creates an attachment from a file on the default storage disk.
- `fromStorageDisk($disk, $path)` — Creates an attachment from a file on a specific storage disk.
- `fromData(Closure $data, $name)` — Creates an attachment from a raw data string returned by the closure.
- `fromUrl($url)` — Creates an attachment by downloading a file from a URL.
- `as($name)` — Sets the display filename for the attachment.
- `withMime($mime)` — Sets the MIME type of the attachment.

**Syntax Rules:**

- `fromData` accepts a closure that returns the raw data bytes. The closure is called at send time, allowing dynamic data generation.
- The `as` and `withMime` methods can be chained in any order.
- Attachable models must implement `Illuminate\Contracts\Mail\Attachable` and define `toMailAttachment()`.
- Inline images are embedded using `$message->embed($path)` or `$message->embedData($data, $name)` in the email template.

**Constraints and Limitations:**

- **Attachments must exist at send time.** If a file is missing or inaccessible, the email will fail.
- **`fromData` closures are executed at send time.** For queued mailables, the closure must be serializable (it cannot capture non-serializable objects like database connections).
- **`fromUrl` downloads the file at send time.** This can slow down sending and may fail if the URL is unreachable.
- **Large attachments may be rejected by mail servers.** Most providers limit attachments to 10–25 MB.

### Annotated Code Examples

**Example 1: Attaching Files from Multiple Sources**

```php
<?php
// File: app/Mail/MonthlyReport.php

namespace App\Mail;

use App\Models\Report;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Attachment;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class MonthlyReport extends Mailable
{
    public function __construct(
        public Report $report,
        public string $pdfContent,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Monthly Report — ' . $this->report->month,
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.reports.monthly',
            with: ['report' => $this->report],
        );
    }

    public function attachments(): array
    {
        return [
            // 1. Attach from a local path with a dynamic filename
            Attachment::fromPath(
                storage_path('app/reports/' . $this->report->id . '.pdf')
            )
                ->as('report-' . $this->report->month . '.pdf')
                ->withMime('application/pdf'),

            // 2. Attach from S3 with a dynamic filename
            Attachment::fromStorageDisk(
                's3',
                'reports/' . $this->report->id . '.xlsx'
            )
                ->as('report-' . $this->report->month . '.xlsx')
                ->withMime('application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'),

            // 3. Attach raw data (generated in memory) with a dynamic name
            Attachment::fromData(
                fn () => $this->pdfContent,
                'summary-' . $this->report->month . '.pdf'
            )
                ->withMime('application/pdf'),
        ];
    }
}
```

```php
// Sending the mailable
$report = Report::find(1);
$pdfContent = generatePdfSummary($report);
Mail::to('stakeholder@example.com')->send(new MonthlyReport($report, $pdfContent));
```

**Expected Output:** The email is sent with three attachments: the PDF from local storage, the Excel file from S3, and the in-memory PDF summary, each with a dynamically generated filename.

**Why This Output Occurs:** The `attachments()` method returns an array of `Attachment` objects. Each attachment is created using a different factory method and configured with `as()` for the filename and `withMime()` for the MIME type. Laravel resolves each attachment at send time — reading from the local path, downloading from S3, and executing the `fromData` closure to generate the in-memory PDF.

---

**Example 2: Attaching an Attachable Model**

```php
<?php
// File: app/Models/Invoice.php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Mailables\Attachment;

class Invoice extends Model implements Attachable
{
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath(
            storage_path('app/invoices/' . $this->id . '.pdf')
        )
            ->as('invoice-' . $this->number . '.pdf')
            ->withMime('application/pdf');
    }
}
```

```php
<?php
// File: app/Mail/InvoiceNotification.php

namespace App\Mail;

use App\Models\Invoice;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class InvoiceNotification extends Mailable
{
    public function __construct(
        public Invoice $invoice,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Invoice #' . $this->invoice->number,
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.invoices.notification',
        );
    }

    public function attachments(): array
    {
        return [
            $this->invoice, // The model implements Attachable
        ];
    }
}
```

**Expected Output:** The invoice PDF is attached to the email with the filename `invoice-INV-2025-001.pdf`.

**Why This Output Occurs:** The `Invoice` model implements the `Attachable` interface and defines `toMailAttachment()`. When the model is included in the `attachments()` array, Laravel calls `toMailAttachment()` to resolve the attachment. This pattern encapsulates the attachment logic within the model, keeping the mailable clean.

### Real-World Cases

- **Invoice attachments:** Attach generated PDF invoices to order confirmation emails, with dynamic filenames based on the invoice number.
- **Report distribution:** Attach Excel or CSV reports from S3 to monthly reporting emails.
- **In-memory PDF generation:** Generate a PDF summary in memory and attach it without writing it to disk.
- **Contract attachments:** Attach signed contract PDFs stored on a cloud disk to notification emails.
- **Image attachments:** Attach user-uploaded photos to notification emails, with the original filename preserved.

---

## 5. Testing Mailables

### Definitions

**Core Definition:** Testing mailables is the process of verifying that the correct mailable classes are sent to the correct recipients with the expected content and attachments, using Laravel's `Mail::fake()` and assertion methods.

**Technical Definition:** The `Illuminate\Support\Testing\Fakes\MailFake` class replaces the real mailer when `Mail::fake()` is called. It records all mailables passed to `send()`, `queue()`, and `later()` without actually sending them. Assertions include `assertSent()`, `assertNotSent()`, `assertQueued()`, `assertNotQueued()`, `assertNothingSent()`, `assertNothingQueued()`, `assertSentCount()`, `assertQueuedCount()`, and `assertOutgoingCount()`. Closures passed to assertion methods receive the mailable instance and can inspect recipients, subject, attachments, and metadata via methods like `hasTo()`, `hasCc()`, `hasSubject()`, and `hasAttachment()`.

**Beginner-Friendly Explanation:** Testing emails means checking that your application sends the right emails to the right people without actually sending them. Laravel's `Mail::fake()` intercepts all emails, so nothing leaves your application during tests. You then assert that a specific mailable was "sent" to a specific address, that it had the right subject, or that it had a particular attachment. This makes email testing fast, reliable, and safe.

### Purposes

- To verify that the correct mailable classes are sent in response to application events.
- To assert that emails are sent to the correct recipients.
- To verify that emails are queued when they implement `ShouldQueue`.
- To inspect email content (subject, attachments, metadata) without sending.
- To prevent accidental email sending during test runs.

### Syntax Rules and Structure

#### Complete General Syntax

```php
use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        // Step 1: Fake the mailer
        Mail::fake();

        // Step 2: Perform the action that sends mail
        $order = Order::factory()->create();
        $this->post('/orders/' . $order->id . '/ship');

        // Step 3: Assert that a mailable was sent
        Mail::assertSent(OrderShipped::class);

        // Assert a mailable was sent twice
        Mail::assertSent(OrderShipped::class, 2);

        // Assert a mailable was sent to a specific address
        Mail::assertSent(OrderShipped::class, 'customer@example.com');

        // Assert a mailable was sent to multiple addresses
        Mail::assertSent(OrderShipped::class, ['customer@example.com', 'admin@example.com']);

        // Assert a mailable was not sent
        Mail::assertNotSent(AnotherMailable::class);

        // Assert a mailable was queued (for ShouldQueue mailables)
        Mail::assertQueued(OrderShipped::class);
        Mail::assertNotQueued(AnotherMailable::class);

        // Assert nothing was sent or queued
        Mail::assertNothingSent();
        Mail::assertNothingQueued();

        // Assert total counts
        Mail::assertSentCount(3);
        Mail::assertQueuedCount(2);
        Mail::assertOutgoingCount(5);
    }
}
```

**Component Breakdown:**

- `Mail::fake()` — Replaces the real mailer with `MailFake`, recording all outgoing mailables.
- `assertSent($mailable, $callback = null)` — Asserts that the given mailable was sent.
- `assertQueued($mailable, $callback = null)` — Asserts that the given mailable was queued.
- `assertNotSent($mailable)` — Asserts that the mailable was not sent.
- `assertNothingSent()` — Asserts that no mailables were sent.
- `assertSentCount($count)` — Asserts the total number of mailables sent.
- `assertOutgoingCount($count)` — Asserts the total number of mailables sent or queued.

**Syntax Rules:**

- `Mail::fake()` must be called before the action that sends mail.
- The first argument to `assertSent` is the mailable class name; the optional second argument is either an address, an array of addresses, a count, or a closure.
- Closures receive the mailable instance and must return `true` for the assertion to pass.
- `assertQueued` should be used for mailables implementing `ShouldQueue`; `assertSent` should be used for synchronous mailables.

**Constraints and Limitations:**

- **`assertSent` will fail silently if used on a queued mailable.** Use `assertQueued` for mailables implementing `ShouldQueue`.
- **`Mail::fake()` does not prevent notifications from being sent.** Use `Notification::fake()` for notifications.
- **Closures receive the mailable instance at the time of sending.** For queued mailables, the instance may be serialised and deserialised.
- **`assertOutgoingCount` counts both sent and queued mailables.**

### Annotated Code Examples

**Example 1: Testing a Mailable Was Sent**

```php
<?php
// File: tests/Feature/OrderShippingTest.php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class OrderShippingTest extends TestCase
{
    use RefreshDatabase;

    public function test_order_shipping_sends_email(): void
    {
        // Step 1: Fake the mailer
        Mail::fake();

        // Step 2: Create an order
        $order = Order::factory()->create([
            'status' => 'processing',
        ]);

        // Step 3: Perform the action that ships the order
        $this->post('/orders/' . $order->id . '/ship');

        // Step 4: Assert the mailable was sent
        Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($order) {
            return $mail->order->id === $order->id
                && $mail->hasTo($order->customer->email)
                && $mail->hasSubject('Order #' . $order->number . ' Shipped');
        });

        // Step 5: Assert the order status was updated
        $this->assertEquals('shipped', $order->fresh()->status);
    }
}
```

**Expected Output:** The test passes if the `OrderShipped` mailable was sent to the order's customer with the correct subject and the order status was updated.

**Why This Output Occurs:** `Mail::fake()` replaces the real mailer with `MailFake`, which records the `OrderShipped` mailable when the controller calls `Mail::to()->send()`. The closure passed to `assertSent` inspects the recorded mailable, checking the order ID, recipient, and subject. If all conditions are met, the assertion passes.

---

**Example 2: Testing Queued Mailables**

```php
<?php
// File: app/Mail/WelcomeEmail.php

namespace App\Mail;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;

class WelcomeEmail extends Mailable implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public User $user,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Welcome to ' . config('app.name'),
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.welcome',
        );
    }
}
```

```php
<?php
// File: tests/Feature/RegistrationTest.php

namespace Tests\Feature;

use App\Mail\WelcomeEmail;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class RegistrationTest extends TestCase
{
    use RefreshDatabase;

    public function test_registration_sends_queued_welcome_email(): void
    {
        // Step 1: Fake the mailer
        Mail::fake();

        // Step 2: Register a new user
        $this->post('/register', [
            'name'                  => 'Jane Doe',
            'email'                 => 'jane@example.com',
            'password'              => 'password',
            'password_confirmation' => 'password',
        ]);

        // Step 3: Assert the mailable was QUEUED, not sent
        Mail::assertQueued(WelcomeEmail::class, function (WelcomeEmail $mail) {
            return $mail->user->email === 'jane@example.com';
        });

        // Step 4: Assert nothing was sent synchronously
        Mail::assertNothingSent();
    }
}
```

**Expected Output:** The test passes if the `WelcomeEmail` was queued (not sent) for the newly registered user.

**Why This Output Occurs:** The `WelcomeEmail` mailable implements `ShouldQueue`, so it is queued rather than sent synchronously. `Mail::assertQueued()` records and inspects queued mailables, while `Mail::assertNothingSent()` verifies that no mailables were sent synchronously. This distinction is critical: using `assertSent` on a queued mailable would fail silently.

### Real-World Cases

- **Registration testing:** Verify that a welcome email is queued when a user registers.
- **Order testing:** Verify that an order confirmation email is sent when an order is placed.
- **Password reset testing:** Verify that a password reset email is sent with the correct reset link.
- **Notification testing:** Verify that multiple recipients receive the correct email.
- **Attachment testing:** Verify that an invoice PDF is attached to an order confirmation email.

---

## References

- Laravel Mail Documentation (Laravel 13.x) — https://laravel.com/docs/13.x/mail 
- Laravel Mail Documentation (Laravel 12.x) — https://laravel.com/docs/12.x/mail 
- Laravel Mail Documentation (Laravel 11.x) — https://laravel.com/docs/11.x/mail 
- Laravel Mail Documentation (Laravel 10.x) — https://laravel.com/docs/10.x/mail 
- Laravel Mailables Documentation — https://laravel.com/docs/13.x/mail#writing-mailables 
- Laravel Markdown Mail Documentation — https://laravel.com/docs/13.x/mail#markdown-mailables 
- Laravel Mail Attachments Documentation — https://laravel.com/docs/13.x/mail#attachments 
- Laravel Mail Testing Documentation — https://laravel.com/docs/13.x/mail#testing-mailables 
- Laravel `Illuminate\Mail\Mailables\Envelope` API — https://api.laravel.com/docs/13.x/Illuminate/Mail/Mailables/Envelope.html 
- Laravel `Illuminate\Mail\Mailables\Content` API — https://api.laravel.com/docs/13.x/Illuminate/Mail/Mailables/Content.html 
- Laravel `Illuminate\Mail\Mailables\Attachment` API — https://api.laravel.com/docs/13.x/Illuminate/Mail/Mailables/Attachment.html 
- Laravel `Illuminate\Contracts\Mail\Attachable` API — https://api.laravel.com/docs/13.x/Illuminate/Contracts/Mail/Attachable.html 
- Laravel Mail Fake API — https://api.laravel.com/docs/13.x/Illuminate/Support/Testing/Fakes/MailFake.html 
- Symfony Mailer Documentation — https://symfony.com/doc/current/mailer.html 
- Mailpit — https://github.com/axllent/mailpit 
- MailHog — https://github.com/mailhog/MailHog 
- Laravel Mail Configuration (Laravel 10.x) — https://laravel.com/docs/10.x/mail#configuration 
- Laravel Failover Configuration — https://laravel.com/docs/10.x/mail#failover-configuration 
- Laravel Round Robin Configuration — https://laravel.com/docs/10.x/mail#round-robin-configuration 
- Laravel Mail Driver Prerequisites — https://laravel.com/docs/10.x/mail#driver-prerequisites 
- Laravel Markdown Mail Components — https://laravel.com/docs/10.x/mail#markdown-mailables 
- Laravel Mail Testing Assertions — https://laravel.com/docs/12.x/mail#testing-mailables 
- Laravel Mail Preview with Log Driver — https://laravel.com/docs/12.x/mail#testing-mailables 
- Mailpit Setup for Laravel — https://github.com/leokhoa/laragon/discussions/1434 
- Local Email Testing in Laravel — https://codecourse.com/articles/local-email-testing-in-laravel