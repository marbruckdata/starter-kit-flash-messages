# starter-kit-flash-messages

## Login response
```php
<?php

namespace App\Http\Responses;

use Laravel\Fortify\Contracts\LoginResponse as LoginResponseContract;
use Laravel\Fortify\Fortify;

class LoginResponse implements LoginResponseContract
{
    /**
     * Create an HTTP response that represents the object.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function toResponse($request)
    {
        return redirect()->intended(Fortify::redirects('login'))->with('toast', 'Welcome back ' . $request->user()->name);
    }
}
```

## Fortify Service Provider
```php
<?php

namespace App\Providers;

use App\Actions\Contracts\UpdatesUserProfilePhoto;
use App\Http\Responses\LoginResponse;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Str;
use Laravel\Fortify\Fortify;

class FortifyServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(
            \Laravel\Fortify\Contracts\LoginResponse::class,
            LoginResponse::class
        );
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        RateLimiter::for('login', function (Request $request) {
            $throttleKey = Str::transliterate(Str::lower($request->input(Fortify::username())).'|'.$request->ip());

            return Limit::perMinute(5)->by($throttleKey);
        });
    }
}
```

## Toast.vue
```javascript
<script setup>
import useToast from '@/composables/useToast'
import { TransitionRoot, TransitionChild } from '@headlessui/vue'

const { body, active, hide } = useToast()
</script>

<template>
    <TransitionRoot as="template" :show="active">
        <TransitionChild
            as="template"
            enter="duration-200 ease-in-out"
            enter-from="-translate-y-full opacity-0"
            leave="duration-200 ease-in-out"
            leave-to="-translate-y-full opacity-0"
        >
            <div class="fixed top-0 left-1/2 -translate-x-1/2 bg-blue-500/90 px-4 py-3 text-white mt-6 z-[100] text-sm">
                {{ body }}
            </div>
        </TransitionChild>
    </TransitionRoot>
</template>
```

