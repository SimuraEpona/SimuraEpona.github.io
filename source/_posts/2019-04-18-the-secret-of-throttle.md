---
title: Laravel API throttle 原理分析
date: 2019-04-18
tags: [LARAVEL, API, Throttle]
---

Laravel 自从5.2版本起就加入了`throttle`中间件来进行限流。下面我们看一下具体的原理是怎样实现的。

<!--more-->

## ThrottleRequests

throttle 中间件的class为`Illuminate\Routing\Middleware\ThrottleRequests`。代码如下：

```
class ThrottleRequests 
{

	 /**
     * The rate limiter instance.
     *
     * @var \Illuminate\Cache\RateLimiter
     */
    protected $limiter;

    /**
     * Create a new request throttler.
     *
     * @param  \Illuminate\Cache\RateLimiter  $limiter
     * @return void
     */
    public function __construct(RateLimiter $limiter)
    {
        $this->limiter = $limiter;
    }

	 ...
}
```

在构造函数中初始化了一个 RateLimiter 类，代码如下：

```php
class RateLimiter
{
    protected $cache;


    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }
}
```

这表示处理限流的相关信息存储位置与我们在项目中配置的 cache driver 一致。接着回到ThrottleRequests中。

```php
// ThrottleRequests
public function handle($request, Closure $next, $maxAttempts = 60, $decayMinutes = 1)
{
    $key = $this->resolveRequestSignature($request);

    $maxAttempts = $this->resolveMaxAttempts($request, $maxAttempts);

    if ($this->limiter->tooManyAttempts($key, $maxAttempts, $decayMinutes)) {
        throw $this->buildException($key, $maxAttempts);
    }

    $this->limiter->hit($key, $decayMinutes);

    $response = $next($request);

    return $this->addHeaders(
        $response, $maxAttempts,
        $this->calculateRemainingAttempts($key, $maxAttempts)
    );
}


protected function resolveRequestSignature($request)
{
    if ($user = $request->user()) {
        return sha1($user->getAuthIdentifier());
    }

    if ($route = $request->route()) {
        return sha1($route->getDomain().'|'.$request->ip());
    }

    throw new RuntimeException(
        'Unable to generate the request signature. Route unavailable.'
    );
}

// RateLimiter
public function tooManyAttempts($key, $maxAttempts, $decayMinutes = 1)
{
    if ($this->attempts($key) >= $maxAttempts) {
        if ($this->cache->has($key.':timer')) {
            return true;
        }

        $this->resetAttempts($key);
    }

    return false;
}

public function hit($key, $decayMinutes = 1)
{
    $this->cache->add(
        $key.':timer', $this->availableAt($decayMinutes * 60), $decayMinutes
    );

    $added = $this->cache->add($key, 0, $decayMinutes);

    $hits = (int) $this->cache->increment($key);

    if (! $added && $hits == 1) {
        $this->cache->put($key, 1, $decayMinutes);
    }

    return $hits;
}
```

和我们熟知的中间件一样，throttle 也通过 handle 方法来进行处理。其中 key 返回的是一个用户标记或者ip标记。即如果用户已经登录，那么根据用户来判断限流，否则根据IP进行限流。接着判断是否达到了限流的上限，如果达到上限抛出异常。否则针对指定的 key 将访问次数加1. 然后在响应中加入指定的 HEADER 后返回。至此，中间件流程基本结束。


## HEADERS

在加入限流中间件之后，会在API中加入特定的HEADER，例如我们设置某个路由的中间件如下：

```php
$api->get('demo', 'xxxxController@xx')->middleware('throttle:5,1');
```

这表示 demo 这个 API 每个用户每1分钟只能访问5次。那么在我们访问的时候会看到在响应中加入了2个新的HEADER。

```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
```

表示路由的API限制次数为5次，还可以访问4次。当我们达到访问上限之后再次访问API，就会得到如下的HEADER：

```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 0
Retry-After: 43
X-RateLimit-Reset: 1555506730
```

新增加的两个HEADER中，Retry-After 表示我们可以在43秒后重新访问，限流的限制将在 X-RateLimit-Reset 返回的时间戳进行重置。


## EXTRA

其实 Laravel 还未我们提供了 ThrottleRequestsWithRedis 类，所实现的功能与前面一致，只不过使用了 Redis 来进行数据存储。