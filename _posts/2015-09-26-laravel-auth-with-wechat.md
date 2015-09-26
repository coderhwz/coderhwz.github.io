---
layout: post
title: 基于Laravel Auth 扩展微信网页授权
keyword: php,__set,__get,array,property
description: PHP 类中get set 数组属性操作
tags: 
    - PHP
    - Laravel
---

# 实现

去掉摸索的过程，实现过程非常快

### routes.php 中设置路由

```php
<?php 

Route::get('oauth2/wechat','Auth\WeChatController@getLogin');

```

### 创建控制器 `WeChatController`


```php
<?php
namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
// 使用Overtrue 优雅的微信SDK
use Overtrue\Wechat\Auth as OAuth2;
use Auth;

class WeChatController extends Controller {

    public function getLogin()
    {
        $appId  = 'appid';
        $secret = 'secret';
        if( !Auth::check() ){
            $auth = new OAuth2($appId, $secret);
            $user = $auth->authorize();
            Auth::attempt($user->all(),false,true);
        }
    }

}

```
控制器中使用了`Auth`，所以我们需要扩展一下`Auth`


### 注册Auth驱动

在`AppServiceProvider` 中注册一个Auth 驱动


```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        Auth::extend('wechat',function(){
            return new UserProvider();
        });

        Auth::setDefaultDriver('wechat');
    }
}

```


### UserProvider
创建`UserProvider`


```php
<?php

namespace App\Providers;

use App\User;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Contracts\Auth\UserProvider as UserContract;


class UserProvider implements UserContract
{


    /**
     * Retrieve a user by their unique identifier.
     *
     * @param  mixed $identifier
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function retrieveById($identifier)
    {
        return User::findOrFail($identifier);
    }

    /**
     * Retrieve a user by their unique identifier and "remember me" token.
     *
     * @param  mixed $identifier
     * @param  string $token
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function retrieveByToken($identifier, $token)
    {
        // TODO: Implement retrieveByToken() method.
    }

    /**
     * Update the "remember me" token for the given user in storage.
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable $user
     * @param  string $token
     * @return void
     */
    public function updateRememberToken(Authenticatable $user, $token)
    {
        // TODO: Implement updateRememberToken() method.
    }

    /**
     * Retrieve a user by the given credentials.
     *
     * @param  array $credentials
     * @return \Illuminate\Contracts\Auth\Authenticatable|null
     */
    public function retrieveByCredentials(array $credentials)
    {
        return User::where('openid','=',$credentials['openid'])->first();
    }

    /**
     * Validate a user against the given credentials.
     *
     * @param  \Illuminate\Contracts\Auth\Authenticatable $user
     * @param  array $credentials
     * @return bool
     */
    public function validateCredentials(Authenticatable $user, array $credentials)
    {
        return true;
    }
}


```

### config
`config/auth.php`中设置`"auth.driver"=>'wechat'`


`app\Http\Middleware\Authenticate.php` 设置授权跳转地址

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Contracts\Auth\Guard;

class Authenticate
{
    /**
     * The Guard implementation.
     *
     * @var Guard
     */
    protected $auth;

    /**
     * Create a new filter instance.
     *
     * @param  Guard  $auth
     * @return void
     */
    public function __construct(Guard $auth)
    {
        $this->auth = $auth;
    }

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($this->auth->guest()) {
            if ($request->ajax()) {
                return response('Unauthorized.', 401);
            } else {
                return redirect()->guest('oauth2/wechat');
            }
        }

        return $next($request);
    }
}

```
ok,就这样
