---
title:  5个让你的开发更加轻松的辅助函数
comments: true
---

> 本文为翻译文章，原文章地址： [5 Laravel Helpers to Make Your Life Easier](https://laravel-news.com/5-laravel-helpers-make-life-easier)

> 在Laravel框架中有许多的辅助函数来帮助开发者更加有效率的进行开发。在这篇文章中，我会列出我个人比较喜欢的5个辅助函数

## data_get()
`data_get()`辅助方法能够让你使用[.]符号来获取数组或者对象中的值。'array_get()'方法也是同样的道理。如果数组或者对象的key不存在的话，这个方法第三个可选参数可以设置一个默认值。

```php
$array = ['albums' => ['rock' => ['count' => 75 ]]];

$count = data_get($array, 'albums.rock.count'); // 75
$avgCost = data_get($array, 'albums.rock.avg_cost', 0); // 0

$object->albums->rock->count = 75;

$count = data_get($object, 'albums.rock.count'); // 75
$avgCost = data_get($object, 'albums.rock.avg_cost', 0); // 0
```

如果在点符号连接中使用通配符'*'将会返回一个数组。

```php
$array = ['albums' => ['rock' => ['count' => 75], 'punk' => ['count' => 12]]];
$counts = data_get($array, 'albums.*.count'); // [75, 12]
```

'data_get()'辅助方法能够让你轻松的再数组或者对象中使用相同的语法来查找数据。这样你就不必检查你之前使用的变量是什么类型了。

## str_plural()
'str_plural()'是将字符串变成对应的复数形式，目前只对英文的单词有效，第二个可选的参数能够让开发者来自己决定返回单数还是复数形式。

```php
str_plural('dog'); // dogs
str_plural('cat'); // cats

str_plural('dog', 2); // dogs
str_plural('cat', 1); // cat

str_plural('child'); // children
str_plural('person'); // people
str_plural('fish'); // fish
str_plural('deer', 2); // deer
```

这个辅助方法最主要的用处就是能够移除类似 `$count == 1 ? 'dog' : 'dogs'` 这样的代码。与之相反的还有一个'str_singular()'的辅助方法。 如果你感兴趣这个方法的工作原理，那么可以看看 [Doctrine’s Inflector Class](https://github.com/doctrine/inflector/blob/master/lib/Doctrine/Common/Inflector/Inflector.php)

## route()
'route()'方法能够生成已经命名的路由，可选的第二个参数将会传递给路由的参数。

```php
Route::get('burgers', 'BurgersController@index')->name('burgers');
route('burgers'); // http://example.com/burgers
route('burgers', ['order_by' => 'price']); // http://example.com/burgers?order_by=price

Route::get('burgers/{id}', 'BurgersController@show')->name('burgers.show');
route('burgers.show', 1); // http://example.com/burgers/1
route('burgers.show', ['id' => 1]); // http://example.com/burgers/1

Route::get('employees/{id}/{name}', 'EmployeesController@show')->name('employees.show');
route('employees.show', [5, 'chris']); // http://example.com/employees/5/chris
route('employees.show', ['id' => 5, 'name' => 'chris']); // http://example.com/employees/5/chris
route('employees.show', ['id' => 5, 'name' => 'chris', 'hide' => 'email']); // http://example.com/employees/5/chris?hide=email
```

如果将第三个可选参数设为false的话，那么将会返回一个相对地址而不是一个绝对地址

'''php
route('burgers.show', 1, false); // /burgers/1
'''

设置了子域名的也是同样的道理， 并且你也可以将Eloquent模型传参给route()方法

```php
Route::domain('{location}.example.com')->group(function () {
    Route::get('employees/{id}/{name}', 'EmployeesController@show')->name('employees.show');
});

route('employees.show', ['location' => 'raleigh', 'id' => 5, 'name' => 'chris']); 


route('burgers.show', Burger::find(1)); // http://example.com/burgers/1
```

## abort_if()
这个辅助方法将会抛出一个异常如果符合满足的要求。第三个可选参数为抛出异常的消息，第四个为header数组。

```php
abort_if(! Auth::user()->isAdmin(), 403);
abort_if(! Auth::user()->isAdmin(), 403, 'Sorry, you are not an admin');
abort_if(Auth::user()->isCustomer(), 403);
```

这个方法最主要的用处就是精简类似下面的代码，通过使用 abort_if() 能够只用一行代码实现同样的功能。

```php
// In "admin" specific controller
public function index()
{
    if (! Auth::user()->isAdmin()) {
        abort(403, 'Sorry, you are not an admin');
    }
}

// better!
public function index()
{
    abort_if(! Auth::user()->isAdmin(), 403);
}
```

> **注意** 如果你是通过这个来控制权限的话，你应该了解一下 [authorization gates](https://d.laravel-china.org/docs/5.5/authorization#gates)。 这样你就可以省去很多的abort检查了。

## optional()
这个方法允许你来获取对象的属性或者调用方法。如果该对象为null，那么属性或者方法也会返回null而不是引起一个错误

```php
// User 1 exists, with account
$user1 = User::find(1);
$accountId = $user1->account->id; // 123

// User 2 exists, without account
$user2 = User::find(2);
$accountId = $user2->account->id; // PHP Error: Trying to get property of non-object

// Fix without optional()
$accountId = $user2->account ? $user2->account->id : null; // null
$accountId = $user2->account->id ?? null; // null

// Fix with optional()
$accountId = optional($user2->account)->id; // null
```

那么，你们最喜欢的辅助函数是哪些呢？