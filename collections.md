# Collections

- [介绍](#introduction)
- [创建集合](#creating-collections)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 介绍

`Illuminate\Support\Collection` 提供流畅方便的工具来操作数组型数据，例如，看下面的代码，我们使用`collect`方法，将数组转化成集合 ，然后对集合中的每一个项执行 `strtoupper`，最后移除所有为空的项：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });


如你所见，`Collection` 类允许你将它的所有方法串连起来以一种流式操作对底动数组执行 mapping 和 reducing 操作。通常，每一个 `Collection` 方法返回一个完整的 `Collection` 实例。


<a name="creating-collections"></a>
## 创建集合

正如上面所提到的，`collect` 方法为给定的数组返回一个新的 `Illuminate\Support\Collection` 实例，所以创建一个集合就是这么简单：

    $collection = collect([1, 2, 3]);

默认情况下，[Eloquent](/docs/{{version}}/eloquent) 模型集合通常 `Collection` 实例的形式返回，然而，请随意在任何方便你的程序的地方使用 `Collection` 类。

<a name="available-methods"></a>
## 可用方法

对于该文档剩余部分，我们将讨论 `Collection` 类上的每一个可用的方法。请记住，所有这些方法都可以串联起来流式地操作底层数组，而且每个一个方法都返回一个新的 `Collection` 实例，允许保存你在必要时保存一份集合的原始拷贝。

你可以从表格中选择任何方法来查看其使用方法的示例：

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">
[all](#method-all)
[chunk](#method-chunk)
[collapse](#method-collapse)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
[each](#method-each)
[filter](#method-filter)
[first](#method-first)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[merge](#method-merge)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereLoose](#method-whereloose)
[zip](#method-zip)
</div>

<a name="method-listing"></a>
## 方法列表

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all` 方法仅返回以集合表示的层底数组：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法根据给定的尺寸将集合拆分为多个更小的集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

当在[视图](/docs/{{version}}/views)使用[Bootstrap](http://getbootstrap.com/css/#grid)这样的网格系统时，这个方法尤其有用，想像一下你需要将一个[Eloquent](/docs/{{version}}/eloquent)模型集合显示到网格中：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法将多个数据合并为一个扁平的集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法用于判断集合是否包含某项：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

你也可以向 `contains` 方法中传入一个键值对，些方法用于判断给定键值对是否存在于集合中：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

最后，你也可以向 `contains` 方法中传入一个回调函数来执行你自己的真值测试：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($key, $value) {
        return $value > 5;
    });

    // false

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法返回集合中项的总数：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法用于比较一个集合跟另外一个集合，或者一个 PHP `array`:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 方法遍历集合中每一项且将每一项传入给定的回调中：

    $collection = $collection->each(function ($item, $key) {
        //
    });

从回调中返回 `false` 退出循环：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法通过给定的回调函数来过滤集合，只保留通过真值过滤的项：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($item) {
        return $item > 2;
    });

    $filtered->all();

    // [3, 4]

`filter` 相对的，请查看[reject](collections#method-reject)方法：

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法返回集合中的第一个通过真值测试的项:

    collect([1, 2, 3, 4])->first(function ($key, $value) {
        return $value > 2;
    });

    // 3

你还可以调用无参的 `first` 方法获取集合中的第一项，如果集合为空，返回 `null`:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法用于将多维集合转化为单维集合：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法用于交换集合中的键与其相应的值：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法根据键从集合中移除项：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // [framework' => 'laravel']

> **注意:** 不像其它大多数集合方法，`forget` 方法不返回新集合，只修改被调用集合

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法根据页码返回一个包含所有应该显示的项的新集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9])->forPage(2, 3);

    $collection->all();

    // [4, 5, 6]

此方法分别需要页码和每页需要显示的项数作为参数。

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法根据给定的 key 获取对应项，如果 key 不存在，返回 `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

你可以选择是否传入一个默认值作为第二个参数：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

你甚至可以传入一个回调作为默认值，如果指定的 key 不存在，则返回此回调的结果：

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy` 方法根据给定的 key 对集合分组：

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

除了传入字符串 `key`，你还可以传入一个回调，改变分组的 key：

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

`has`用于判断一个给定的 key 是否存在于集合中：

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    // false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法用于合并集合中的对象，其参数取决于集合中对象的类型。

如果集合包含数组和对象，你应该传入你想合并属性的 key, 以及值之间的连接符:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

如果集合仅包含字符串或者数字，只用向方法传入连接符这一个参数：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect` 方法用于删除任何不存在于给定 `array` 或集合中的值：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

如你所见，结果集合将保留原集合的 key。

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

如果集合为空时，`isEmpty` 方法返回 `true`，否则，返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

根据给定的 key 索引集合：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'desk'],
        ['product_id' => 'prod-200', 'name' => 'chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

如果多个对象具有相同的 key, 只有最后一个会出现在新的集合中。

你还可以传入你自己的回调，返回索引集合的 key：

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */


<a name="method-keys"></a>
#### `keys()` {#collection-method}

`keys` 方法返回集合中所有的 keys: 

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法返回集合最一个通过给定测试的元素：

    collect([1, 2, 3, 4])->last(function ($key, $value) {
        return $value < 3;
    });

    // 2

你还可以调用无参的 `last` 方法获取集合中的最后一个元素，如果集合为空，返回 `null`：

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

`map` 遍历整个集合且将每个一个值传入给定的回调，这个回调函数可以自由修改对象并返回，因此而返回一个包含已修改对象的新集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> **注意:** 跟其它大多数集合方法一样，`map` 方法返回一个新的集合实例而不会修改集合，如果想转换原始集合，可以使用[`transform`](#method-transform)。

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge` 将给定的数组合并为一个集合，数组中任何字符串 key 与集合中 key 匹配时，将覆盖集合中的值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $merged = $collection->merge(['price' => 100, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

如果给定的数组的 key 都是数字，对应的值将会添加到集合的最后面：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck` 方法根据给定的 key 检索出集合中所有相应的值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

你还可以指定结果集合中的 key 是怎么样的：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop` 方法移除并返回集合中的最后一项：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend` 添加对象到集合的开始位置：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull` 根据给定的 Key 从集合中移除并返回对象:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push` 方法添加对象到集合的末尾：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put` 方法设置给定的 key 和 value 到集合中：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法从集合中随机返回一个对象：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

你还可以传入一个整数给 `random`，如果这个整数比 `1` 要大，将返回一个集合：

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce` 方法将集合归约为单个值，将每次递归的结果传给一下递归：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

首轮递归的时 `$carry` 的值是 `null`， 然而，你可通过传入第二个参数来指定他的初始值:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject` 利用回调函数过滤集合，当回调函数返回 `true` 时表示将结果集中移除该项：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($item) {
        return $item > 2;
    });

    $filtered->all();

    // [1, 2]

与 `reject` 相反的操作，请查看[`filter`](#method-filter) 方法。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reverse` 方法将集合反序：

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

`search` 查找给定的 value，如果找到则返回它的 key, 如果对象未找到，返回 `false`：

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

搜索默认使用的是「宽松」比对，要使用严格比对，传入 `true` 作为第二个参数：

    $collection->search('4', true);

    // false

另外，你可以传入自己的回调函数来搜索通过测试逻辑的对象：

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

`shift` 方法从集合中删除且返回第一项：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

`shuffle` 方法随机将集合中的对象打乱：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] // (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice` 方法从一个给定的序号开始返回集合的一个切片：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果你想限制返回的切片的尺寸，传入一个尺寸值作为第二个参数：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

返回的切片将拥有新的数字索引 Key，如果你想保留原始的 key, 传入 `true` 作为第三个参数：

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort` 对集合排序：

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

经过排序的集合保留原来的数组 key，在这个例子中，我们使用[`values`](#method-values) 方法重置 Key 为连续的数字索引：

给嵌套的数组或对象集合排序，请查看[`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc)方法。

如果你有更高级的排序要求，可以向 `sort` 方法传入包含自己算法的回调函数，请查看 PHP 文档[`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)，这个就是集合的 `sort` 方法的底层调用函数。

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy` 方法根据给定的 key 为集合排序：

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

排好序的集合保留原始数组的 key，在这个例子中，我们使用[`values`](#method-values) 方法将 key 重置为连续的数字索引。

你还可以传入你自己的回调函数来决定如何为集合的值排序：

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

这个方法跟[`sortBy`](#method-sortby)方法具有相同的签名，但是会以想的试给集合排序。

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice` 方法移除并返回集合的一部分，从给定的序号开始：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

你可以传入第二个参数来限制返回部分的尺寸:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

除此之外，你可以传入第三个参数包含一个新的值来替换从集合中移除的项：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum` 返回集合中所有项的和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含嵌套的数组或对象，你应该传入一个 key 来决定对哪个值求和：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

除此，你可以传入你自己的回调来决定集合中的哪些值需要求和：

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

`take` 方法返回包含指定数量对象的集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你还可以传入负整数表示获取从集合末尾开始的指定数量的对象：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray` 方法将集合转化为一般的 PHP `array`，如果集合的值是[Eloquent](/docs/{{version}}/eloquent) 模型，模型也将被转化为数组：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **注意:** `toArray` 也会将其嵌套的对象转化为数组，如果你想获取原本的底层数组，使用[`all`](#method-all)。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson` 方法将集合转换成 JSON：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

The `transform` method iterates over the collection and calls the given callback with each item in the collection. The items in the collection will be replaced by the values returned by the callback:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> **Note:** 不像其它大多数集合方法，`transform` 修改被调用的集合，如果你希望创建一个新的集合，请使用[`map`](#method-map)。

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 返回集合中所有的唯一的对象：

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

返回的集合保留原始的数组 key，在这个例子中，我们使用[`values`](#method-values)方法将这些 key 重置为连续的数字索引。

当处理嵌套的数组和对象时，你可以指定用于判断唯一性的 key:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

你还可以传入你自己的回调函数，判断对象的唯一性：

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

<a name="method-values"></a>
#### `values()` {#collection-method}

`values` 方法返回一个新的集合，key 重置为连续的整数：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */
<a name="method-where"></a>
#### `where()` {#collection-method}

`where` 方法根据给定的 key / value 对过滤集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
    */

`where` 在检查对象值时使用严格比对，[`whereLoose`](#where-loose)使用「宽松」比对。

<a name="method-whereloose"></a>
#### `whereLoose()` {#collection-method}

这个方法与[`where`](#method-where)方法具有相同的签名，然后对所有值使用「宽松」比对。

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法根据集合中值，以相应的位置，合并给定数组的值：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]
