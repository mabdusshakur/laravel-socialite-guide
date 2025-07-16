# Laravel 12 Socialite Social Login Integration Guide

This guide covers integrating Laravel Socialite for social login with all major providers (Google, Facebook, GitHub, Twitter, LinkedIn, Bitbucket, GitLab, etc.) in Laravel 12.

---

## 1. Install Laravel Socialite

```bash
composer require laravel/socialite
```

---

## 2. Configure Socialite in `config/services.php`

Add your credentials for each provider:

```php
// config/services.php

return [
    // ...existing code...
    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => env('GITHUB_REDIRECT_URI'),
    ],
    'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_CLIENT_SECRET'),
        'redirect' => env('GOOGLE_REDIRECT_URI'),
    ],
    'facebook' => [
        'client_id' => env('FACEBOOK_CLIENT_ID'),
        'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
        'redirect' => env('FACEBOOK_REDIRECT_URI'),
    ],
    'twitter' => [
        'client_id' => env('TWITTER_CLIENT_ID'),
        'client_secret' => env('TWITTER_CLIENT_SECRET'),
        'redirect' => env('TWITTER_REDIRECT_URI'),
    ],
    'linkedin' => [
        'client_id' => env('LINKEDIN_CLIENT_ID'),
        'client_secret' => env('LINKEDIN_CLIENT_SECRET'),
        'redirect' => env('LINKEDIN_REDIRECT_URI'),
    ],
    'bitbucket' => [
        'client_id' => env('BITBUCKET_CLIENT_ID'),
        'client_secret' => env('BITBUCKET_CLIENT_SECRET'),
        'redirect' => env('BITBUCKET_REDIRECT_URI'),
    ],
    'gitlab' => [
        'client_id' => env('GITLAB_CLIENT_ID'),
        'client_secret' => env('GITLAB_CLIENT_SECRET'),
        'redirect' => env('GITLAB_REDIRECT_URI'),
    ],
];
```

---

## 3. Add Environment Variables

Add the following to your `.env` file:

```env
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_REDIRECT_URI=https://your-app.com/auth/github/callback

GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URI=https://your-app.com/auth/google/callback

FACEBOOK_CLIENT_ID=your-facebook-client-id
FACEBOOK_CLIENT_SECRET=your-facebook-client-secret
FACEBOOK_REDIRECT_URI=https://your-app.com/auth/facebook/callback

TWITTER_CLIENT_ID=your-twitter-client-id
TWITTER_CLIENT_SECRET=your-twitter-client-secret
TWITTER_REDIRECT_URI=https://your-app.com/auth/twitter/callback

LINKEDIN_CLIENT_ID=your-linkedin-client-id
LINKEDIN_CLIENT_SECRET=your-linkedin-client-secret
LINKEDIN_REDIRECT_URI=https://your-app.com/auth/linkedin/callback

BITBUCKET_CLIENT_ID=your-bitbucket-client-id
BITBUCKET_CLIENT_SECRET=your-bitbucket-client-secret
BITBUCKET_REDIRECT_URI=https://your-app.com/auth/bitbucket/callback

GITLAB_CLIENT_ID=your-gitlab-client-id
GITLAB_CLIENT_SECRET=your-gitlab-client-secret
GITLAB_REDIRECT_URI=https://your-app.com/auth/gitlab/callback
```

---

## 4. Add Socialite Routes

```php
// routes/web.php

use App\Http\Controllers\Auth\SocialiteController;

Route::get('auth/{provider}', [SocialiteController::class, 'redirect'])->name('socialite.redirect');
Route::get('auth/{provider}/callback', [SocialiteController::class, 'callback'])->name('socialite.callback');
```

---

## 5. Create Socialite Controller

```php
// app/Http/Controllers/Auth/SocialiteController.php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Laravel\Socialite\Facades\Socialite;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Str;

class SocialiteController extends Controller
{
    public function redirect($provider)
    {
        return Socialite::driver($provider)->redirect();
    }

    public function callback($provider)
    {
        $socialUser = Socialite::driver($provider)->user();

        $user = User::firstOrCreate([
            'email' => $socialUser->getEmail(),
        ], [
            'name' => $socialUser->getName() ?? $socialUser->getNickname() ?? 'User',
            'password' => bcrypt(Str::random(24)),
            'provider' => $provider,
            'provider_id' => $socialUser->getId(),
        ]);

        Auth::login($user);

        return redirect('/home');
    }
}
```

---

## 6. Update User Model (Optional)

Add provider fields if you want to store them:

```php
// database/migrations/xxxx_xx_xx_add_provider_to_users_table.php

public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('provider')->nullable();
        $table->string('provider_id')->nullable();
    });
}
```

And in `User.php`:

```php
// app/Models/User.php

protected $fillable = [
    'name', 'email', 'password', 'provider', 'provider_id',
];
```

---

## 7. Add Social Login Buttons to Blade View

```blade
<!-- resources/views/auth/login.blade.php -->
<a href="{{ route('socialite.redirect', 'google') }}">Login with Google</a>
<a href="{{ route('socialite.redirect', 'github') }}">Login with GitHub</a>
<a href="{{ route('socialite.redirect', 'facebook') }}">Login with Facebook</a>
<a href="{{ route('socialite.redirect', 'twitter') }}">Login with Twitter</a>
<a href="{{ route('socialite.redirect', 'linkedin') }}">Login with LinkedIn</a>
<a href="{{ route('socialite.redirect', 'bitbucket') }}">Login with Bitbucket</a>
<a href="{{ route('socialite.redirect', 'gitlab') }}">Login with GitLab</a>
```

---

## 8. Notes

- Register your app with each provider to get client IDs and secrets.
- Some providers (e.g., Twitter) may require additional configuration or use Socialite community drivers.
- For more providers, see: [Socialite Providers](https://socialiteproviders.com/)
- Always secure your callback URLs and validate user data.

---

## 9. References

- [Laravel Socialite Docs](https://laravel.com/docs/12.x/socialite)
- [Socialite Providers](https://socialiteproviders.com/)
