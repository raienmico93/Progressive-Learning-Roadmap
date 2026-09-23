# Modern Ecosystem Integration (Starter Kits & Social Auth): A Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Modern ecosystem integration in Laravel encompasses the official starter kits (Breeze, Jetstream), headless authentication backends (Fortify), social OAuth2 sign-on (Socialite), and forward-looking security features (MFA/2FA and Passkeys) that together provide a complete, production-ready authentication layer.

**Technical Definition:** Laravel's authentication ecosystem is built on a modular architecture where the core `AuthManager` is wrapped by higher-level packages. Laravel Fortify provides a headless, backend-only authentication implementation including routes, controllers, and actions for login, registration, password reset, email verification, and two-factor authentication. Laravel Breeze and Jetstream are frontend starter kits that consume Fortify (or implement equivalent logic) and publish views/controllers into the application. Laravel Socialite is an OAuth2 client library that abstracts provider-specific authentication flows. Passkeys support is provided via `laravel/passkeys` (server-side WebAuthn) and `@laravel/passkeys` (frontend helpers), integrated into Fortify via `Features::passkeys()`.

**Beginner-Friendly Explanation:** Laravel gives you several "starter packs" for authentication. Breeze is the simple one—it sets up login and registration pages you can customize. Jetstream is the full-featured one—it includes team management and two-factor auth out of the box. Fortify is the "engine" underneath—it handles all the backend logic but leaves the design to you. Socialite lets users log in with Google, GitHub, or other services. And Passkeys let users log in with their fingerprint or face instead of a password.

### Key Characteristics

- **Layered Architecture:** Fortify provides the backend; starter kits provide the frontend.
- **Stack Flexibility:** Breeze supports Blade, React, Vue, and API-only; Jetstream supports Livewire and Inertia/Vue.
- **Social OAuth2:** Socialite abstracts provider-specific OAuth flows with a consistent API.
- **Account Linking:** Social accounts can be linked to existing users to prevent duplicates.
- **2FA Ready:** Fortify includes TOTP-based two-factor authentication with recovery codes.
- **Passkeys Native:** Laravel now ships first-class WebAuthn support via `laravel/passkeys` and Fortify integration.
- **Feature Flags:** Fortify uses `Features::` to enable/disable specific authentication capabilities.

### Prerequisites

- A Laravel application with the `users` table and `User` model.
- Composer for package installation.
- For Socialite: OAuth credentials from the provider (client ID, client secret, redirect URL).
- For 2FA/Passkeys: HTTPS in production (WebAuthn requires secure context).
- For Jetstream: Livewire or Inertia/Vue stack preference.

### Related Programming Areas

- **Laravel Sanctum:** API token and SPA authentication.
- **Laravel Passport:** Full OAuth2 server for third-party integrations.
- **Laravel Fortify:** Headless authentication backend.
- **Laravel Socialite:** OAuth2 client for social login.
- **WebAuthn:** Browser-based cryptographic authentication standard.

### Core Concepts / Features

1. Starter Kits Comparison (Breeze vs. Jetstream vs. Fortify)
2. Socialite (OAuth2 Social Sign-On)
3. Multi-Factor Authentication (MFA/2FA)
4. Passkeys (WebAuthn)

---

## 1. Laravel Breeze & Jetstream: Starter Kit Comparison

### Definitions

**Core Definition:** Laravel starter kits are pre-built authentication scaffolds that provide ready-to-use login, registration, password reset, and profile management interfaces, differing in complexity, feature set, and frontend stack.

**Technical Definition:** Laravel Breeze is a minimal starter kit that publishes controllers, routes, and views directly into the application, supporting Blade with Alpine, Livewire, React, and Vue (Inertia). Laravel Jetstream is a more advanced starter kit built on top of Fortify, offering Livewire or Inertia/Vue stacks with additional features including team management, two-factor authentication, browser session management, and API token management via Sanctum. Laravel Fortify is the headless backend that powers Jetstream and can be used independently with custom UIs.

**Beginner-Friendly Explanation:** Breeze is like buying a basic furniture kit—you get the essentials (login, registration) and can customize everything. Jetstream is like buying a fully furnished room—it comes with extras like team management and 2FA, but there's more to learn and customize. Fortify is the engine that powers Jetstream—you can use it alone if you want to build your own interface from scratch.

### Purposes

- To provide pre-built authentication UI for rapid application development.
- To offer different complexity levels matching project requirements.
- To support multiple frontend stacks (Blade, React, Vue, Livewire).
- To separate backend authentication logic (Fortify) from frontend presentation (Breeze/Jetstream).
- To enable quick prototyping with customizable, published code.

### Syntax Rules and Structure

**Complete General Syntax — Breeze Installation:**

```bash
# Fresh Laravel project with Breeze
laravel new my-app
cd my-app
composer require laravel/breeze --dev
php artisan breeze:install
# Choose stack: blade, livewire, react, vue, api
php artisan migrate
npm install && npm run dev
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `composer require laravel/breeze --dev` | Installs Breeze as a dev dependency. |
| `php artisan breeze:install` | Runs the interactive installer. |
| Stack choice | `blade`, `livewire`, `react`, `vue`, or `api`. |

**Complete General Syntax — Jetstream Installation:**

```bash
composer require laravel/jetstream
php artisan jetstream:install livewire --teams
# Or: php artisan jetstream:install inertia --teams
php artisan migrate
npm install && npm run build
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `--teams` | Enables team management features. |
| Stack choice | `livewire` or `inertia` (Vue). |
| Underlying | Powered by Laravel Fortify. |

**Complete General Syntax — Fortify Installation:**

```bash
composer require laravel/fortify
php artisan fortify:install
php artisan migrate
```

```php
// config/fortify.php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `php artisan fortify:install` | Publishes config, actions, and migrations. |
| `Features::` | Enables/disables specific authentication features. |
| Headless | Provides routes/controllers but no views. |

**Syntax Rules:**
- Starter kits must be installed on a **fresh** Laravel project (they overwrite files) .
- Breeze publishes editable controllers, routes, and views into `app/` and `resources/` .
- Jetstream uses Fortify under the hood but adds UI, teams, and session management .
- Fortify requires you to define views via `Fortify::loginView()`, `Fortify::registerView()`, etc. .

**Constraints and Limitations:**
- **Published code is unversioned:** Breeze/Jetstream publish code that you own and must update manually when Laravel conventions change .
- **No React in Jetstream:** Jetstream only supports Livewire and Inertia/Vue .
- **Breeze is legacy:** Laravel 12 introduced new React/Vue/Livewire starter kits; Breeze is maintained but not advertised .
- **Fortify is headless:** You must build all views yourself .

### Multiple Annotated Complete Code Examples

**Example 1: Breeze with Blade Stack**

```bash
laravel new my-app
cd my-app
composer require laravel/breeze --dev
php artisan breeze:install
# Select: blade with Alpine
php artisan migrate
npm install && npm run dev
```

**Expected Output:** The application has functional login, registration, password reset, email verification, and profile pages. Routes are in `routes/auth.php`, controllers in `app/Http/Controllers/Auth/`, and views in `resources/views/auth/`.

**Why:** Breeze publishes editable code directly into the application. You can open `LoginController.php` and modify it. This is ideal for developers who want full control over the authentication UI .

---

**Example 2: Jetstream with Teams and 2FA**

```bash
laravel new my-app
cd my-app
composer require laravel/jetstream
php artisan jetstream:install livewire --teams
php artisan migrate
npm install && npm run build
```

**Expected Output:** The application includes login, registration, 2FA setup/verification, team creation and management, browser session management, and API token management.

**Why:** Jetstream provides a complete feature set for SaaS applications. The `--teams` flag enables team management. Fortify handles the backend logic; Jetstream provides the UI .

---

**Example 3: Fortify with Custom Views**

```php
<?php
// app/Providers/FortifyServiceProvider.php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Fortify\Fortify;

class FortifyServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Fortify::loginView(fn () => view('auth.login'));
        Fortify::registerView(fn () => view('auth.register'));
        Fortify::requestPasswordResetLinkView(fn () => view('auth.forgot-password'));
        Fortify::resetPasswordView(fn ($request) => view('auth.reset-password', ['request' => $request]));
        Fortify::verifyEmailView(fn () => view('auth.verify-email'));
        Fortify::confirmPasswordView(fn () => view('auth.confirm-password'));
        Fortify::twoFactorChallengeView(fn () => view('auth.two-factor-challenge'));
    }
}
```

**Expected Output:** Fortify registers all authentication routes and controllers. The views are rendered from your custom Blade templates.

**Why:** Fortify is headless—it provides the backend routes and logic, but you define the views. This is ideal when you have a custom design system or are building a decoupled SPA .

### Real-World Cases

- **Small Projects / MVPs:** Breeze with Blade for quick setup and full control .
- **SaaS Applications:** Jetstream with Livewire for teams, 2FA, and session management out of the box .
- **Custom Design Systems:** Fortify alone with custom React/Vue components for maximum flexibility .
- **API-Only Backends:** Breeze with `api` stack or Fortify with Sanctum for SPA/mobile clients .

### References

- Laravel Breeze vs Jetstream vs Fortify (LinkedIn) — https://www.linkedin.com/posts/saqib-hussain-babbar-99ab56287_laravel-authentication-systems-breeze-vs-activity-7425070704234184705-BUSW
- Laravel 12 Starter Kits Guide (Laravel Daily) — https://laraveldaily.com/post/laravel-12-starter-kits-options-guide-choose
- Starter Kits and Using Laravel Breeze (Laravel Daily) — https://laraveldaily.com/lesson/laravel-from-scratch/starter-kits-breeze
- Authentication in Laravel with Breeze (Kinsta) — https://kinsta.com/es/blog/laravel-breeze/
- Laravel Fortify Documentation (13.x) — https://laravel.com/docs/13.x/fortify

---

## 2. Socialite: OAuth2 Social Sign-On

### Definitions

**Core Definition:** Laravel Socialite is an OAuth2 client library that simplifies authentication with external providers (Google, GitHub, Apple, etc.), abstracting provider-specific flows into a consistent, fluent API.

**Technical Definition:** Socialite provides `Socialite::driver($provider)->redirect()` for initiating the OAuth flow and `Socialite::driver($provider)->user()` for retrieving the authenticated user. It supports built-in providers (Facebook, X, LinkedIn, Google, GitHub, GitLab, Bitbucket, Slack) and over 100 community adapters via `socialiteproviders.com`. User data is returned as a `Laravel\Socialite\Contracts\User` instance with `getId()`, `getName()`, `getEmail()`, and `getAvatar()` methods.

**Beginner-Friendly Explanation:** Socialite lets users click "Log in with Google" instead of filling out a registration form. Your app redirects to Google, the user approves access, and Google sends them back with their profile information. Socialite handles all the OAuth complexity so you just call `->user()` to get their email and name.

### Purposes

- To allow users to authenticate using existing social accounts (Google, GitHub, etc.).
- To reduce registration friction and improve conversion rates.
- To retrieve user profile data (name, email, avatar) from providers.
- To link social accounts to existing users for unified account management.
- To support enterprise SSO via generic OIDC providers (Okta, Auth0, Keycloak).

### Syntax Rules and Structure

**Complete General Syntax — Installation:**

```bash
composer require laravel/socialite
```

**Complete General Syntax — Configuration:**

```php
// config/services.php
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URI'),
],

'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => env('GITHUB_REDIRECT_URI'),
],
```

**Complete General Syntax — Controller:**

```php
use Laravel\Socialite\Facades\Socialite;

// Redirect to provider
Route::get('/auth/{provider}/redirect', function ($provider) {
    return Socialite::driver($provider)->redirect();
});

// Handle callback
Route::get('/auth/{provider}/callback', function ($provider) {
    $socialUser = Socialite::driver($provider)->user();

    // $socialUser->getId(), getName(), getEmail(), getAvatar()

    $user = User::updateOrCreate([
        'email' => $socialUser->getEmail(),
    ], [
        'name' => $socialUser->getName(),
        'email_verified_at' => now(),
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Socialite::driver('google')` | Creates a driver instance for the provider. |
| `->redirect()` | Returns a redirect response to the provider. |
| `->user()` | Returns the authenticated user from the callback. |
| `getId()`, `getEmail()` | Provider-specific user attributes. |

**Complete General Syntax — Database Schema:**

```sql
social_accounts
├── id
├── user_id (FK → users)
├── provider (google, github, etc.)
├── provider_id (unique ID from provider)
├── token (encrypted access token)
├── refresh_token (encrypted)
├── expires_at
├── avatar_url
└── created_at / updated_at

UNIQUE(provider, provider_id)
```

**Syntax Rules:**
- Provider credentials are configured in `config/services.php` .
- The `redirect` URL must match the callback URL registered with the provider exactly .
- The `user()` method retrieves the authenticated user from the OAuth callback.
- Tokens should be stored **encrypted** if persisted .
- The `password` column should be nullable for users who only authenticate via social login .

**Constraints and Limitations:**
- **Callback URL match:** The redirect URL in your config must match the provider's registered callback exactly (including protocol and trailing slashes) .
- **Email availability:** Not all providers return email addresses (e.g., GitHub may require specific scopes) .
- **Token storage:** Provider tokens should be encrypted and hidden from serialization .
- **Stateless providers:** Some providers (e.g., SAML2) require stateless authentication, which cannot link to existing accounts .

### Multiple Annotated Complete Code Examples

**Example 1: Google OAuth Login**

```php
<?php
// routes/web.php

use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/google/redirect', function () {
    return Socialite::driver('google')->redirect();
});

Route::get('/auth/google/callback', function () {
    $googleUser = Socialite::driver('google')->user();

    // Find or create user
    $user = User::updateOrCreate(
        ['email' => $googleUser->getEmail()],
        [
            'name' => $googleUser->getName(),
            'email_verified_at' => now(),
            'avatar' => $googleUser->getAvatar(),
        ]
    );

    Auth::login($user);

    return redirect('/dashboard');
});
```

**Expected Output:** Clicking "Login with Google" redirects to Google's consent screen. After approval, the user is redirected back, logged in, and sent to the dashboard.

**Why:** `Socialite::driver('google')->redirect()` initiates the OAuth flow. The callback retrieves the user via `->user()`. `updateOrCreate()` prevents duplicate accounts .

---

**Example 2: Account Linking (Authenticated User)**

```php
<?php
// Link a social account to an existing user
Route::get('/auth/{provider}/link', function ($provider) {
    return Socialite::driver($provider)->redirect();
})->middleware('auth');

Route::get('/auth/{provider}/link/callback', function ($provider) {
    $socialUser = Socialite::driver($provider)->user();

    // Link to the currently authenticated user
    auth()->user()->socialAccounts()->updateOrCreate(
        ['provider' => $provider],
        [
            'provider_id' => $socialUser->getId(),
            'token' => encrypt($socialUser->token),
            'avatar_url' => $socialUser->getAvatar(),
        ]
    );

    return redirect('/profile')->with('status', 'Account linked!');
})->middleware('auth');
```

**Expected Output:** The authenticated user's account is linked to their social provider. Future logins via that provider authenticate as the same user.

**Why:** Account linking prevents duplicate accounts. The `socialAccounts` relationship stores the provider, provider ID, and encrypted token .

---

**Example 3: Generic OIDC for Enterprise SSO**

```php
<?php
// config/services.php
'oidc' => [
    'client_id' => env('OIDC_CLIENT_ID'),
    'client_secret' => env('OIDC_CLIENT_SECRET'),
    'redirect' => env('OIDC_REDIRECT_URI'),
    'issuer' => env('OIDC_ISSUER_URL'), // e.g., https://your-org.okta.com
],
```

```php
// Using socialiteproviders/oidc package
Route::get('/auth/oidc/redirect', function () {
    return Socialite::driver('oidc')->redirect();
});

Route::get('/auth/oidc/callback', function () {
    $user = Socialite::driver('oidc')->user();

    // User data from OIDC claims: sub, email, name, picture
    // ...
});
```

**Expected Output:** Users authenticate via enterprise identity providers (Okta, Auth0, Keycloak) using standard OIDC.

**Why:** The `socialiteproviders/oidc` package enables generic OIDC support with auto-discovery via `.well-known/openid-configuration` .

### Real-World Cases

- **Consumer Applications:** "Login with Google/GitHub" for frictionless onboarding .
- **Enterprise SSO:** Generic OIDC for corporate identity providers (Okta, Auth0) .
- **Account Linking:** Users connect multiple social accounts to one profile .
- **Mobile Apps:** Socialite with Sanctum for API token authentication after social login .

### References

- Laravel Socialite Documentation (12.x) — https://laravel.com/docs/12.x/socialite
- SSO Provider Integration ADR (GitHub) — https://github.com/Sourdough-start/sourdough/blob/master/docs/adr/003-sso-provider-integration.md
- Laravel Socialite Guide (FPT Polytechnic) — https://caodang.fpt.edu.vn/tin-tuc-poly/huong-dan-su-dung-laravel-socialite-dang-nhap-xa-hoi-de-de-dang-cho-ung-dung-web.html
- Top PHP Laravel Authentication Libraries 2026 (Medium) — https://medium.com/@php.nerd/top-php-laravel-authentication-libraries-options-for-2026-the-complete-decision-guide-e4c68039bf2e
- OAuth Documentation (Statamic) — https://raw.githubusercontent.com/statamic/docs/refs/heads/6.x/content/collections/pages/oauth.md

---

## 3. Multi-Factor Authentication (MFA/2FA)

### Definitions

**Core Definition:** Multi-factor authentication (MFA), often called two-factor authentication (2FA), is a security mechanism that requires users to provide two or more verification factors to gain access to their account.

**Technical Definition:** Laravel's 2FA implementation is provided through Laravel Fortify, which supports TOTP (Time-based One-Time Password) via authenticator apps (Google Authenticator, Authy). Fortify handles the backend logic: generating secrets, QR codes, recovery codes, and verifying TOTP codes. The `TwoFactorAuthenticatable` trait on the `User` model provides methods like `twoFactorQrCodeSvg()`, `confirmTwoFactorAuthentication()`, and `recoveryCodes()`. The `two-factor` middleware protects routes requiring 2FA challenge completion.

**Beginner-Friendly Explanation:** After entering your password, Laravel asks for a 6-digit code from your phone. You open Google Authenticator, type the code, and you're logged in. Even if someone steals your password, they can't get in without your phone. Fortify handles all the technical details—generating the secret, creating the QR code for setup, and verifying the codes.

### Purposes

- To add a second layer of security beyond passwords.
- To protect against credential theft and unauthorized access.
- To provide TOTP-based authentication via authenticator apps.
- To generate and manage recovery codes for account recovery.
- To allow users to enable/disable 2FA from their profile settings.

### Syntax Rules and Structure

**Complete General Syntax — Fortify Configuration:**

```php
// config/fortify.php
'features' => [
    // ...
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
        // 'window' => 0, // TOTP window (seconds)
    ]),
],
```

**Complete General Syntax — User Model:**

```php
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use TwoFactorAuthenticatable;
}
```

**Complete General Syntax — Routes:**

```php
// Fortify automatically registers:
// GET  /user/two-factor-qr-code
// GET  /user/two-factor-recovery-codes
// POST /user/two-factor-authentication
// POST /user/confirmed-two-factor-authentication
// DELETE /user/two-factor-authentication
// GET  /user/two-factor-challenge
// POST /user/two-factor-challenge
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Features::twoFactorAuthentication()` | Enables 2FA features. |
| `TwoFactorAuthenticatable` trait | Adds 2FA methods to the User model. |
| `confirm` | Requires confirmation before enabling. |
| `confirmPassword` | Requires password confirmation to manage 2FA. |

**Complete General Syntax — Middleware Protection:**

```php
Route::middleware(['auth', 'two-factor'])->group(function () {
    Route::get('/dashboard', ...);
});
```

**Syntax Rules:**
- Fortify must be installed and configured with 2FA enabled .
- The `User` model must use the `TwoFactorAuthenticatable` trait .
- Users must confirm 2FA setup by entering a valid TOTP code .
- Recovery codes are generated automatically and should be saved by the user .

**Constraints and Limitations:**
- **Fortify required:** 2FA is not available in Breeze out of the box; it requires Fortify or Jetstream .
- **HTTPS recommended:** TOTP codes are time-sensitive; ensure server time is accurate.
- **Recovery codes:** Users must save recovery codes; losing them and the authenticator device can lock out the account .
- **Rate limiting:** Failed 2FA attempts should be rate-limited to prevent brute force .

### Multiple Annotated Complete Code Examples

**Example 1: Enabling 2FA with Fortify**

```php
<?php
// app/Http/Controllers/TwoFactorController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TwoFactorController extends Controller
{
    public function enable(Request $request)
    {
        // Fortify handles the actual logic via its routes
        // This is typically handled by the Fortify action
        return redirect()->route('two-factor.show');
    }
}
```

```blade
{{-- resources/views/settings/two-factor.blade.php --}}
@if (Laravel\Fortify\Features::canManageTwoFactorAuthentication())
    <div>
        <h3>Two-Factor Authentication</h3>
        @if (auth()->user()->two_factor_secret)
            <p>Two-factor authentication is enabled.</p>
            <form method="POST" action="/user/two-factor-authentication">
                @csrf
                @method('DELETE')
                <button type="submit">Disable 2FA</button>
            </form>
        @else
            <p>Two-factor authentication is disabled.</p>
            <form method="POST" action="/user/two-factor-authentication">
                @csrf
                <button type="submit">Enable 2FA</button>
            </form>
        @endif
    </div>
@endif
```

**Expected Output:** Users can enable/disable 2FA from their settings page. When enabled, a QR code is displayed for scanning with an authenticator app.

**Why:** Fortify provides the routes (`POST /user/two-factor-authentication`) and logic. The Blade view conditionally displays the appropriate form based on `two_factor_secret` .

---

**Example 2: 2FA Challenge During Login**

```php
<?php
// Fortify automatically handles the 2FA challenge
// After successful password authentication, if 2FA is enabled,
// the user is redirected to /user/two-factor-challenge

// The challenge view:
```

```blade
{{-- resources/views/auth/two-factor-challenge.blade.php --}}
<form method="POST" action="/user/two-factor-challenge">
    @csrf
    <div>
        <label for="code">Authentication Code</label>
        <input id="code" type="text" name="code" inputmode="numeric" autofocus>
    </div>
    <button type="submit">Verify</button>
</form>
```

**Expected Output:** After entering correct credentials, users with 2FA enabled see the challenge page. Entering a valid TOTP code completes login.

**Why:** Fortify intercepts the login flow and redirects to the 2FA challenge when `two_factor_secret` is set .

---

**Example 3: Requiring 2FA for Admin Users**

```php
<?php
// app/Policies/AdminPolicy.php

namespace App\Policies;

use App\Models\User;
use Illuminate\Auth\Access\Response;

class AdminPolicy
{
    public function accessAdmin(User $user): Response
    {
        if ($user->isAdmin() && !$user->hasEnabledTwoFactorAuthentication()) {
            return Response::deny('Admins must enable two-factor authentication.');
        }

        return Response::allow();
    }
}
```

**Expected Output:** Admin users who haven't enabled 2FA are denied access to admin routes with a clear message.

**Why:** The policy checks both admin status and 2FA enrollment, enforcing the security requirement .

### Real-World Cases

- **Admin Panels:** Require 2FA for all administrative users .
- **Financial Applications:** Mandatory 2FA for all users due to regulatory requirements.
- **Enterprise SaaS:** Optional or enforced 2FA based on organization policy.
- **Consumer Apps:** Optional 2FA for security-conscious users .

### References

- Two-Factor Authentication in Laravel: Packages and Options (Laravel Daily) — https://laraveldaily.com/post/two-factor-auth-laravel-packages-tools-methods
- Laravel Fortify Documentation (13.x) — https://laravel.com/docs/13.x/fortify
- Two-Factor-Laravel Package (Packagist) — https://packagist.org/packages/emargareten/two-factor-laravel
- Laravel 2FA Package (Packagist) — https://packagist.org/packages/makelarisjr/laravel-2fa
- Laravel 2FA Package (Mortogo321) — https://packagist.org/packages/mortogo321/laravel-2fa

---

## 4. Passkeys (WebAuthn)

### Definitions

**Core Definition:** Passkeys are a passwordless authentication method based on the WebAuthn standard, using cryptographic key pairs stored on the user's device (e.g., fingerprint, Face ID, hardware security key) instead of passwords.

**Technical Definition:** Laravel's passkey support is provided by `laravel/passkeys` (server-side WebAuthn implementation) and `@laravel/passkeys` (frontend helpers for React, Vue, Svelte). Fortify integrates passkeys via `Features::passkeys()`, and the `User` model must implement `PasskeyUser` and use the `PasskeyAuthenticatable` trait. Passkeys are registered per-device and can be managed from the user's security settings.

**Beginner-Friendly Explanation:** Instead of typing a password, you use your fingerprint or face to log in. Your device creates a cryptographic key pair—the private key stays on your device, and the public key is stored on the server. When you log in, your device signs a challenge with the private key, and the server verifies it with the public key. No password is ever sent over the internet.

### Purposes

- To provide passwordless authentication using biometrics or hardware keys.
- To eliminate password-related security risks (phishing, credential stuffing).
- To improve login convenience (no password to remember).
- To support cross-device authentication via synced passkeys (iCloud Keychain, Google Password Manager).
- To future-proof authentication as the industry moves toward passwordless standards.

### Syntax Rules and Structure

**Complete General Syntax — User Model:**

```php
use Laravel\Fortify\Contracts\PasskeyUser;
use Laravel\Fortify\PasskeyAuthenticatable;

class User extends Authenticatable implements PasskeyUser
{
    use PasskeyAuthenticatable;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `implements PasskeyUser` | Marks the model as passkey-capable. |
| `use PasskeyAuthenticatable` | Adds passkey methods to the model. |

**Complete General Syntax — Fortify Configuration:**

```php
// config/fortify.php
'features' => [
    // ...
    Features::passkeys(),
],
```

**Complete General Syntax — Frontend Integration:**

```javascript
// Using @laravel/passkeys (React, Vue, Svelte helpers)
import { registerPasskey, authenticatePasskey } from '@laravel/passkeys';
```

**Complete General Syntax — Middleware:**

```php
// Passkeys are handled by Fortify's routes
// No additional middleware required for basic functionality
```

**Syntax Rules:**
- Fortify must be installed and configured with `Features::passkeys()` enabled .
- The `User` model must implement `PasskeyUser` and use `PasskeyAuthenticatable` .
- Passkeys require HTTPS in production (WebAuthn requires secure context).
- Frontend helpers are available for React, Vue, and Svelte .

**Constraints and Limitations:**
- **Browser support:** WebAuthn is supported in modern browsers (Chrome, Safari, Firefox, Edge).
- **Device dependency:** Passkeys are stored on devices; cross-device use requires syncing (iCloud Keychain, Google Password Manager) or hardware security keys.
- **HTTPS required:** WebAuthn requires a secure context (HTTPS or localhost).
- **Fortify dependency:** Passkeys are integrated via Fortify; standalone use requires manual WebAuthn implementation .

### Multiple Annotated Complete Code Examples

**Example 1: Enabling Passkeys in Fortify**

```php
<?php
// config/fortify.php

use Laravel\Fortify\Features;

return [
    'features' => [
        Features::registration(),
        Features::resetPasswords(),
        Features::emailVerification(),
        Features::updateProfileInformation(),
        Features::updatePasswords(),
        Features::twoFactorAuthentication([
            'confirm' => true,
            'confirmPassword' => true,
        ]),
        Features::passkeys(), // Enable passkey support
    ],
];
```

**Expected Output:** Fortify registers passkey registration and authentication routes. Users can register passkeys from their security settings.

**Why:** `Features::passkeys()` enables the passkey feature set in Fortify .

---

**Example 2: User Model Setup for Passkeys**

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\Contracts\PasskeyUser;
use Laravel\Fortify\PasskeyAuthenticatable;

class User extends Authenticatable implements PasskeyUser
{
    use Notifiable, PasskeyAuthenticatable;
}
```

```bash
# Publish and run the passkey migration
php artisan vendor:publish --tag=fortify-migrations
php artisan migrate
```

**Expected Output:** The `users` table is updated with passkey-related columns (or a separate `passkeys` table is created).

**Why:** The `PasskeyAuthenticatable` trait provides methods for registering and verifying passkeys. The migration adds the necessary database columns .

---

**Example 3: Nova Passkey Authentication**

```php
<?php
// app/Providers/NovaServiceProvider.php

namespace App\Providers;

use Laravel\Fortify\Features;
use Laravel\Nova\Nova;
use Laravel\Nova\NovaApplicationServiceProvider;

class NovaServiceProvider extends NovaApplicationServiceProvider
{
    protected function fortify(): void
    {
        // Enable passkeys for Nova
        Features::passkeys();
    }
}
```

**Expected Output:** Nova users can access a new "User Security" page from the user menu to manage passkeys.

**Why:** Nova integrates with Fortify's passkey feature, providing a UI for passkey management .

### Real-World Cases

- **Consumer Applications:** "Sign in with fingerprint" for mobile and desktop apps.
- **Enterprise:** Hardware security keys (YubiKey) for high-security access.
- **Cross-Device:** Passkeys synced via iCloud Keychain or Google Password Manager.
- **Future-Proofing:** Preparing for a passwordless future as industry adoption grows .

### References

- Laravel April Product Updates (Laravel Blog) — https://laravel.com/blog/laravel-april-product-updates
- Laravel Nova Authentication — https://nova.laravel.com/docs/v5/customization/authentication
- Laravel Fortify Documentation (13.x) — https://laravel.com/docs/13.x/fortify

---

## References

- Laravel Breeze vs Jetstream vs Fortify (LinkedIn) — https://www.linkedin.com/posts/saqib-hussain-babbar-99ab56287_laravel-authentication-systems-breeze-vs-activity-7425070704234184705-BUSW
- Laravel 12 Starter Kits Guide (Laravel Daily) — https://laraveldaily.com/post/laravel-12-starter-kits-options-guide-choose
- Starter Kits and Using Laravel Breeze (Laravel Daily) — https://laraveldaily.com/lesson/laravel-from-scratch/starter-kits-breeze
- Authentication in Laravel with Breeze (Kinsta) — https://kinsta.com/es/blog/laravel-breeze/
- Laravel Fortify Documentation (13.x) — https://laravel.com/docs/13.x/fortify
- Two-Factor Authentication in Laravel: Packages and Options (Laravel Daily) — https://laraveldaily.com/post/two-factor-auth-laravel-packages-tools-methods
- Laravel Socialite Documentation (12.x) — https://laravel.com/docs/12.x/socialite
- SSO Provider Integration ADR (GitHub) — https://github.com/Sourdough-start/sourdough/blob/master/docs/adr/003-sso-provider-integration.md
- Laravel Socialite Guide (FPT Polytechnic) — https://caodang.fpt.edu.vn/tin-tuc-poly/huong-dan-su-dung-laravel-socialite-dang-nhap-xa-hoi-de-de-dang-cho-ung-dung-web.html
- Laravel April Product Updates (Laravel Blog) — https://laravel.com/blog/laravel-april-product-updates
- Laravel Nova Authentication — https://nova.laravel.com/docs/v5/customization/authentication
- Top PHP Laravel Authentication Libraries 2026 (Medium) — https://medium.com/@php.nerd/top-php-laravel-authentication-libraries-options-for-2026-the-complete-decision-guide-e4c68039bf2e
- Laravel Auth Systems Comparison (LobeHub) — https://lobehub.com/skills/fusengine-agents-laravel-auth
- OAuth Documentation (Statamic) — https://raw.githubusercontent.com/statamic/docs/refs/heads/6.x/content/collections/pages/oauth.md
- Two-Factor-Laravel Package (Packagist) — https://packagist.org/packages/emargareten/two-factor-laravel
- Laravel 2FA Package (Packagist) — https://packagist.org/packages/makelarisjr/laravel-2fa
- Laravel 2FA Package (Mortogo321) — https://packagist.org/packages/mortogo321/laravel-2fa
- iOS App - Login With Laravel (Stack Overflow) — https://stackoverflow.com/questions/73823959/ios-app-login-with-laravel-api-passport-or-sanctum