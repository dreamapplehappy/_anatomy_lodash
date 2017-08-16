---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# 前言

这个文档主要用来讲解[Lodash](https://github.com/lodash/lodash)的源码，我们从零开始构建一个完整的Lodash，项目的地址是[sharp-lodash](https://github.com/dreamapplehappy/sharp-lodash)
然后有什么不足的地方大家可以在这里提出来[Issues](https://github.com/dreamapplehappy/sharp-lodash/issues)，
关于本网站的问题可以在[_anatomy_lodash](https://github.com/dreamapplehappy/_anatomy_lodash)提出来；
阅读源码的版本是`4.17.4`


# Array(数组)

## _.slice 

```javascript
/**
 * Creates a slice of `array` from `start` up to, but not including, `end`.
 *
 * **Note:** This method is used instead of
 * [`Array#slice`](https://mdn.io/Array/slice) to ensure dense arrays are
 * returned.
 *
 * @since 3.0.0
 * @category Array
 * @param {Array} array The array to slice.
 * @param {number} [start=0] The start position.
 * @param {number} [end=array.length] The end position.
 * @returns {Array} Returns the slice of `array`.
 */
function slice(array, start, end) {
  // #1  
  let length = array == null ? 0 : array.length
  if (!length) {
    return []
  }
  // #2
  start = start == null ? 0 : start
  end = end === undefined ? length : end
    
  // #3  
  if (start < 0) {
    start = -start > length ? 0 : (length + start)
  }
  end = end > length ? length : end
  if (end < 0) {
    end += length
  }
  // #4
  length = start > end ? 0 : ((end - start) >>> 0)
  start >>>= 0

  // #5 
  let index = -1
  const result = new Array(length)
  while (++index < length) {
    result[index] = array[index + start]
  }
  return result
}

export default slice
```
首先我们来说一下这个函数的作用，它的作用就是获取一个数组的切片；所谓切片，就是指数组的一部分连续元素，当然也可以是数组的全部元素。我们这时可能想到了数组本身就有一个[`slice`][3]方法，那我们为什么不使用原生的数组的那个`slice`方法而非要自己重新写一个呢？

有两个原因：

  - 更好的兼容性，确保了IE浏览器在版本小于9的情况下，对于元素节点列表的操作可以返回一个[密集的数组(dense-arrays,这个不太好翻译)][4]
  - 比原生的方法效率更高，这个会在本文的后面有一个对比图。

下面我们就来好好看一下这个函数，首先这个函数需要接收三个参数，但是后两个参数不是必须选择的；第一个参数是一个数组，可以是元素的节点集合；第二个参数表示开始截取切片的位置，第三个参数表示的是切片截取的截至位置，但是不包含这个数所在位置的元素。

接下来是分步骤的讲解，我在相应的位置做了标记，大家看的时候可以找标记的位置，下面的讲解就是按照标记的位置来的。

  - `#1`：我们使用了三目运算符来判断是否传入了一个数组，如果没有传入数组我们直接把数组的长度设置为0；反之，我们就获取数组的长度；然后做了一个判断，如果数组的长度为0，我们直接返回一个空的数组。
  - `#2`：判断参数`start`和`end`是否存在；如果都存在的话，就取传入的这个值；如果不存在的话，`start`的取值默认为`0`, `end`的取值默认为数组的长度。
  - `#3`：判断参数`start`是否是负数；如果`start`是负数的话，再比较一下`start`的相反数与数组长度的大小，如果大于数组的长度，那么就赋值为0；反之，就把`start`赋值为`length + start`，也就是从数组的后面开始数开始截取的位置；然后判断一下`end`是否大于数组的长度，如果大于数组的长度，那么就把它赋值为数组的长度；然后判断一下`end`是否小于`0`，如果小于`0`的话，就赋值为`end + length`，也就是从后向前数结束的位置。
  - `#4`：我们看到`>>>`这样一个操作符，这个是按位移动操作符，表示`向右无符号移动`；我们先来看一下代码，首先判断`start`是否大于`end`，如果大于`end`就把`length`的值设为`0`，否则就把`end`减去`start`然后向右无符号移动`零位`；然后把`start`向右无符号移动零位。那么这里为什么要使用`>>>`这个按位操作符呢？**首先我们要了解[`>>>`][5]的作用，`>>>`的作用就是把一个数字，变成一个无符号的32位的整数，那么`num >>> 0`的作用，就是把`num`变成一个无符号的32位的整数，不论`num`是负数还是小数。而且我们还需要知道，JavaScript的数组的最大长度是`2^32-1`，所以这样做也避免了数组的索引超出界限。**
  - `#5`：上一步计算出了我们要取的数组的长度，然后我们在这一步就新创建了一个数组，然后将我们要获取的数组的值，从原数组中拷贝过来；然后返回这个数组。

到这里，我们已经把这个函数需要注意的地方都讲解了一下；那么接下来就需要我们自己去实现这么一个函数了，[slice][6]是我实现的一个版本。大家可以去好好练一下啦，没有什么特别困难的地方。

对了，上面我们说了要比较一下`_.slice`和原生的`[].slice`方法的性能，下图是在我的电脑上的一个测试，大家也可以自己测试测试一下，测试的链接是[slice-vs-slice][7]

![_.slice vs native slice][8]

从上图可以明显的看到，`_.slice`方法比原生的`[].slice`方法性能要好很多。

[1]:https://lodash.com/
[2]:https://github.com/lodash/lodash/blob/master/slice.js
[3]:https://mdn.io/Array/slice
[4]:http://2ality.com/2012/06/dense-arrays.html
[5]:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators 
[6]:https://runkit.com/dreamapplehappy/slice
[7]:https://jsperf.com/slice-vs-slice
[8]:https://dreamapple.me/2017/08/13/Lodash%E6%BA%90%E7%A0%81%E8%AE%B2%E8%A7%A3-1/slice-vs-slice.png

## _.chunk 

函数内部依赖的函数如下：

函数名字 | 源码 | 函数简介
--------- | ----------- | -----------
slice | [slice.js](https://github.com/lodash/lodash/blob/master/slice.js) | xxx


# Collection(集合)

# Date(日期)

# Function(函数)

# Lang(语言)

# Math(数学)

# Number(数字)

# Object(对象)

# Seq

# String(字符串)

# Util(工具)

# Properties(特性)

# Methods(方法)

# Authentication

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint retrieves a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

