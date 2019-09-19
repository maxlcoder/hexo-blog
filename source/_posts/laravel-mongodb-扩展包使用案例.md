---
title: laravel mongodb 扩展包使用案例
date: 2019-09-19 09:50:03
tags:
---


> 本文使用 `jenssegers/mongodb` 做案例演示，  
> 大部分的使用方式和 `SQL` 类的使用类似，这里列举一些常见的用法，和一些比较特殊的用法

1. 添加

```php
$res = DB::collection('users')->insert([
    'name' => '里斯',
    'age' => 18
]);
dump($res); // true/false

$res = DB::collection('users')->insertGetId([
    'name' => '里斯3',
    'age' => 18
]);
dump($res);
echo($res);
/*
返回一个对象，注意这个对象默认实现了 __toString() 方法，所以当把它当作字符串输出时，默认会返回对应的 oid 的值

dump($res)
MongoDB\BSON\ObjectId {
    "oid": "5d826a911225e615fd455ce2"
}

echo $res;
5d7fd428c5adc6c7ffe6e3bd
*/
```

2. 更新

```php
DB::collection('users')
    ->where('name', '里斯')
    ->update([
        'age' => 20,
    ]); // 返回影响的行数

// upsert 更新时如果找不到则创建
DB::collection('users')
    ->where('name', 'Lily')
    ->update(['name' => 'Lily2'], ['upsert' => true]);; // 返回影响的行数

// 向指定的 collection 插入 array 值
DB::collection('users')->where('name', 'nihao')->push('items', 'boots'); //返回影响的行数
// 向指定的 collection 插入不重复的 array 值
DB::collection('users')->where('name', 'John')->push('items', 'boots', true); // 返回影响的行数
DB::collection('users')->where('name', 'John')->push('messages', ['from' => 'Jane Doe', 'message' => 'Hi John']);

// 移除指定 collection 中属性的匹配的值
DB::collection('users')->where('name', 'John')->pull('items', 'boots'); // 返回影响的行数
DB::collection('users')->where('name', 'John')->pull('messages', ['from' => 'Jane Doe', 'message' => 'Hi John']);

// 移除一个 collection 中的某个属性
DB::collection('users')->where('name', 'John')->unset('note');
DB::collection('users')->where('name', 'John')->drop('note');
DB::collection('users')->where('name', 'John')->drop(['note', 'tag']);
```

3. 查询

```php
// 单条
DB::collection('users')
    ->where('age', '>', 1)
    ->first(); // 返回数组

// 多条
$users = DB::collection('users')
    ->where('age', '>', 1)
    ->get();
$users->dump();

/*
这里返回一个 Collection ，实质可以使用到 Laravel Collection 的常用一些操作
Illuminate\Support\Collection^ {#732
  #items: array:2 [
    0 => array:4 [
      "_id" => MongoDB\BSON\ObjectId {#731
        +"oid": "5d7fd428c5adc6c7ffe6e3bd"
      }
      "name" => "nihao"
      "age" => 14.0
      "status" => "A"
    ]
    1 => array:2 [
      "_id" => MongoDB\BSON\ObjectId {#735
        +"oid": "5d826e555cc80979b28fbcf3"
      }
      "name" => "Lily2"
    ]
  ]
}
*/

// 返回指定的列，默认都会附加 _id
DB::collection('users')
    ->where('age', '>', 1)
    ->first(['name']);
DB::collection('users')
    ->where('age', '>', 1)
    ->get(['name'])


/* 
MongoDB 查询数据时默认不是直接返回全部的结果，而是采用 cursor 的形式，遍历结果集的时候，会逐步的取出数据，其中存在一个 cursor 过期时间(cursor timeout)，默认是 10 分钟，也就是 10 分钟内，没有去取出数据，MongoDB 会将游标收回
可以使用如下操作来，禁用过期时间 (不推荐)
*/
DB::collection('users')->timeout(-1)->get();

// project，可以指定 $slice 来返回基于结果的数量范围内的值，指定 $elemMatch 返回匹配的值
DB::collection('users')->project(['name' => ['$slice' => 1]])->get();

// 分页
$limit = 25;
$projections = ['id', 'name'];
$users = DB::collection('users')->paginate($limit, $projections);
dump($users);

/*
输出：
Illuminate\Pagination\LengthAwarePaginator {#741
  #total: 2
  #lastPage: 1
  #items: Illuminate\Support\Collection {#734
    #items: array:2 [
      0 => array:2 [
        "_id" => MongoDB\BSON\ObjectId {#736
          +"oid": "5d7fd428c5adc6c7ffe6e3bd"
        }
        "name" => "nihao"
      ]
      1 => array:2 [
        "_id" => MongoDB\BSON\ObjectId {#735
          +"oid": "5d826e555cc80979b28fbcf3"
        }
        "name" => "Lily2"
      ]
    ]
  }
  #perPage: 25
  #currentPage: 1
  #path: "http://localhost"
  #query: []
  #fragment: null
  #pageName: "page"
  +onEachSide: 3
  #options: array:2 [
    "path" => "http://localhost"
    "pageName" => "page"
  ]
}
*/

// 缓存
$users = User::remember(10)->get()
```

4. 删除
```php
DB::collection('users')->where('name', 'nihao')->delete(); // 返回影响的行数
DB::collection('users')->delete('$id -> xxxxxxxxxx'); // 返回影响的行数

```

> 其他的用法可以参考具体的 `Builder` 进行查阅验证