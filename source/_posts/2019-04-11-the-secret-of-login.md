---
title: Laravel 登录原理剖析
date: 2019-04-11
tags: [LARAVEL, AUTH, PHP]
---

## 简介

Laravel 中实现用户认证非常简单。实际上，几乎所有东西都已经为你配置好了。其配置文件位于 `config/auth.php`，其中包含了用于调整认证服务行为的注释清晰的选项配置。

其核心是由 Laravel 的认证组件的「看守器」和「提供器」组成。看守器定义了该如何认证每个请求中用户。例如，Laravel 自带的 session 看守器会使用 session 存储和 cookies 来维护状态。

提供器中定义了该如何从持久化的存储数据中检索用户。Laravel 自带支持使用 Eloquent 和数据库查询构造器来检索用户。当然，你可以根据需要自定义其他提供器。

不过对大多数应用而言，可能永远都不需要修改默认身份认证配置。

<!--more-->

> 上面的简介出自 [Laravel-China](https://learnku.com/docs/laravel/5.5/authentication/1308) 社区的文档中。

除此之外，Laravel还提供了一个简单的命令来快速生成身份验证所需的路由和视图：

```bash
php artisan make:auth
```

## 原理剖析

### Auth::routes()

`make:auth`命令在`routes/web.php`中插入了下面的代码：

```php
Auth::routes();
```

其中，Auth 是使用 Facades 来调用的。那么我们可以在文件`vendor/laravel/framework/src/Illuminate/Support/Facades/Auth.php`中找到`routes`方法。

```php
// Laravel5.8 中的 routes 方法
public static function routes(array $options = [])
{
    static::$app->make('router')->auth($options);
}

// Laravel5.5 中的 routes 方法
public static function routes()
{
    static::$app->make('router')->auth();
}
```

在这个方法中，调用了`vendor/laravel/framework/src/Illuminate/Routing/Router.php`文件中的 auth 方法。

```php
// Laravel5.8 中的 auth 方法
public function auth(array $options = [])
{
    // Authentication Routes...
    $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
    $this->post('login', 'Auth\LoginController@login');
    $this->post('logout', 'Auth\LoginController@logout')->name('logout');

    // Registration Routes...
    if ($options['register'] ?? true) {
        $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
        $this->post('register', 'Auth\RegisterController@register');
    }

    // Password Reset Routes...
    if ($options['reset'] ?? true) {
        $this->resetPassword();
    }

    // Email Verification Routes...
    if ($options['verify'] ?? false) {
        $this->emailVerification();
    }
}

// Laravel 5.5 中的 auth 方法
public function auth()
{
    // Authentication Routes...
    $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
    $this->post('login', 'Auth\LoginController@login');
    $this->post('logout', 'Auth\LoginController@logout')->name('logout');

    // Registration Routes...
    $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
    $this->post('register', 'Auth\RegisterController@register');

    // Password Reset Routes...
    $this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
    $this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
    $this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
    $this->post('password/reset', 'Auth\ResetPasswordController@reset');
}
```

从上面的代码中我们可以看到，auth 方法为我们注册了登录、登出、密码重设等路由。而在 Laravel5.8 中我们则可以在`Auth::routes()`中传入参数来控制个别路由是否注册，比如：

```php
Auth::routes([
    'register' => false, 
    'reset' => false, 
    'verify' => true
]);
```
可以禁用注册和重置路由，启用邮箱验证路由。

### 登录原理

在`LoginController`中引入了`Illuminate\Foundation\Auth\AuthenticatesUsers`这个 trait。登录的逻辑使用了其中的 login 方法。trait 文件中的相关代码如下：

```php
public function login(Request $request)
{
    $this->validateLogin($request);

    if ($this->hasTooManyLoginAttempts($request)) {
        $this->fireLockoutEvent($request);

        return $this->sendLockoutResponse($request);
    }

    if ($this->attemptLogin($request)) {
        return $this->sendLoginResponse($request);
    }

    $this->incrementLoginAttempts($request);

    return $this->sendFailedLoginResponse($request);
}
```

具体逻辑为首先验证表单提交字段是否通过验证，即`$this->validateLogin($request)`方法，下面的第一个 if 代码块用来判断是否超过登录次数限制。接着判断用户能否进行登录。即`$this->attemptLogin($request)`方法。

```php
protected function attemptLogin(Request $request)
{
    return $this->guard()->attempt(
        $this->credentials($request), $request->filled('remember')
    );
}

protected function guard()
{
    return Auth::guard();
}

protected function credentials(Request $request)
{
    return $request->only($this->username(), 'password');
}
```

通过上面的代码可以看到，我们调用 Auth::guard() 来判断用户能否登录，如果认证通过那么用户登录成功，否则登录失败。至于这个 Guard 的工作原理，我们下面详细说明。


## Auth::guard()->attempt()

### AuthManager

由于 Auth 的 Facades 对应的底层类为`Illuminate\Auth\AuthManager`，因此我们首先分析这个类。

```php
public function guard($name = null)
{
    $name = $name ?: $this->getDefaultDriver();

    return $this->guards[$name] ?? $this->guards[$name] = $this->resolve($name);
}

public function getDefaultDriver()
{
    return $this->app['config']['auth.defaults.guard'];
}
```

在登录的逻辑中，由于没有传入特定的参数，因为我们将会调用默认的 Driver 和 Provider。即`config/auth.php`中的defaults 配置：

```php
'defaults' => [
    'guard' => 'web',
    'passwords' => 'users',
],

...

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

   ...
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\User::class,
    ],
],

'passwords' => [
    'users' => [
        'provider' => 'users',
        'table' => 'password_resets',
        'expire' => 60,
    ],
],

```
通过配置文件可以看到，我们使用的 driver 是 session driver 和 eloquent provider。在 createSessionDriver 方法中我们新建了一个SessionGuard。
因此 Auth::guard() 返回的是一个 SessionGuard 类。

在 SessionGuard 中我们可以看到有 attempt 方法。

```php
public function attempt(array $credentials = [], $remember = false)
{
    $this->fireAttemptEvent($credentials, $remember);

    $this->lastAttempted = $user = $this->provider->retrieveByCredentials($credentials);


    if ($this->hasValidCredentials($user, $credentials)) {
        $this->login($user, $remember);

        return true;
    }

    $this->fireFailedEvent($user, $credentials);

    return false;
}

protected function hasValidCredentials($user, $credentials)
{
    return ! is_null($user) && $this->provider->validateCredentials($user, $credentials);
}

public function login(AuthenticatableContract $user, $remember = false)
{
    $this->updateSession($user->getAuthIdentifier());

    if ($remember) {
        $this->ensureRememberTokenIsSet($user);

        $this->queueRecallerCookie($user);
    }

    $this->fireLoginEvent($user, $remember);

    $this->setUser($user);
}
```

在这里我们可以看到首先触发 attempt 事件。接着我们通过 EloquentUserProvider 中的retrieveByCredentials 得到除 password 字段外匹配的 User 模型。如果匹配成功，则判断密码是否一致，如果一致，则登录成功。否则失败。

> 注：密码校验默认使用 bcrypt。

EloquentUserProvider 中相关代码如下：

```php
public function retrieveByCredentials(array $credentials)
{
    if (empty($credentials) ||
       (count($credentials) === 1 &&
        array_key_exists('password', $credentials))) {
        return;
    }

    $query = $this->newModelQuery();

    foreach ($credentials as $key => $value) {
        if (Str::contains($key, 'password')) {
            continue;
        }

        if (is_array($value) || $value instanceof Arrayable) {
            $query->whereIn($key, $value);
        } else {
            $query->where($key, $value);
        }
    }

    return $query->first();
}

public function validateCredentials(UserContract $user, array $credentials)
{
    $plain = $credentials['password'];

    return $this->hasher->check($plain, $user->getAuthPassword());
}

```

这基本上就是登录过程的原理。