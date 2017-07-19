+++
categories = ["PHP"]
date = "2017-07-18T21:24:02+08:00"
isCJKLanguage = true
tags = ["Illuminate", "Database"]
title = "Illuminate Database 使用过程中遇到的一些问题记录"

+++

程序员自己总想造轮子, 自己造的当然也能用, 只是在很多方面还是没有那些流行的库考虑全面, 诸如方便调试, 灵活性和扩展性等. 之前因为没有找到一个自认为很好的数据库操作类库, 就自己写了一个, 用起来没有问题, 但开始没有考虑到上面提到的这些方面, 当要解决的问题更加复杂, 就越发的感觉力不从心. 正好项目组要我重新组织一个新的 PHP 框架出来供整个部门所有PHP 项目使用, 就花了些时间比较了一下目前比较流行的方案. 看了一堆觉得还是 Laravel 的功能比较全面, 虽然现在在某些方面已经跑偏了, 不过瑕不掩瑜, 它仍然是一个很优秀的数据库操作工具.

配置方面不多说, 这里仅仅说一些比较常遇到的问题. 首先声明, 我仅仅研究了 QueryBuilder, 而不是 Eloquent, 因为我们组所有人都不太喜欢标准的 ORM 的方式, 觉得直接操作实体不习惯.

1. 每张表对应一个类, 这个类包含表名, 表中所有的列名还有基本的操作方法

对于没有分库分表的情形, 在类中直接指定表名即可, 如果有分库分表的需求, 肯定需要有专门的 hash 方法来决定哪种数据应该放在哪个库哪张表里, 本质上还是一样, 这里不做过多阐述. 

关于定义一系列常量用于表示列名, 其实是避免重复写字符串出错. 在项目中用到相应列名的地方统一用常量表示, 可以减少拼写出错的概率, 同时也方便统一修改, 我们在开发过程中也确实遇到了需要修改列名的情形, 直接修改常量的值即可.

我觉得最重要的是在这一层到底需要做到何种程度的封装, 比如我一张用户信息表, 里面可能包含用户 id, 姓名, 邮箱, 地址等信息, 而有一个 api 是希望通过用户的 id 查询地址, 那么你是在这层提供一个类型`queryAddressById($id) : string` 方法呢还是直接`queryOne($where): array`方法呢? 我觉得是后者. 因为如果在这层里提供的全是诸如前一种方法, 那这个类将会变得非常难以维护, 后期有各种查询需求你可能都要重新写一个方法来支持. 并且可能查询的方式多重多样, 比如这次是用 id 查询, 下次可能用邮箱查, 为了避免不必要的麻烦, 我决定在这里只提供最基础的几个方法.

```php

<?php
...
public function query($where, $page, $perPage);
public function count($where);
public function queryOne($where);
public function create(array $row);
public function update($where, array $row);
public function delete($where);
```

这几个方法已经能满足80%以上的需求. 对于不同的表, 这几个方法的实现大同小异, 我想过把这些方法写到一个 trait 里面供所有的类使用, 但后来发现这样虽然可以解耦, 也就是表对应的类只需要关心表名和列名, 不用关心操作方法, 但会大大减小灵活性. 比如有些表我可能会在用户传进来的所有查询条件后再加一个条件, 例如是否已删除的标记, 而这个标记(列)只在特定的表才有, 无法公用. 所以如果不存在这种情形, 大可以放心的使用 trait.

2. query 方法返回的数据是 Collection, 怎么办?

其实 Collection 在大多数情况下用起来和普通的 array 没什么不同, 只是有时返回空 Collection 时如果你把它当做数组, 用`empty($collection)`判断肯定会出错. Collection 其实在某种程度上大大的提高了对数组操作的灵活性, 而且现在很多项目都自己实现了 Collection, 这也说明了 PHP 确实应该内置这种类型了, 或者 psr 是否应该制定一个标准供大家使用呢?

3. 既然第1条提到了统一的查询条件, 那怎样才能统一呢?

这个问题很重要. 首先明确使用场景, 多数情况下我们的所有查询条件都是"且"的关系, 解释成 SQL 语句就是`where xxx = 'xx' and yyy = 'yy'`这种, 当然也存在特例, 出现或的情况. 我们大家都爱 PHP 的数组, 所以我还是想从数组出发, 制定一个写查询条件的数组格式, 然后写一个 formatter 来将它统一处理成Illuminate Database 能接受的**闭包形式**的 where.

这样做是因为闭包适合做统一处理.

假设查询条件是 attribute = 2 and age > 20, 我就可以写成`[['attribute', '=', '2'], ['age', '>', 20]]`. 这样就可以写一个 builder 把它处理成 QueryBuilder .

```php
// rev 01
<?php
public function build(array $conditions)
{
    return function (Builder $builder) use ($conditions) {
        if (empty($conditions)) {
            return $builder;
        }

        foreach ($conditions as $condition) {
            $builder->where($condition[0], $condition[1], $condition[2]);
        }

        return $builder;
    }
} 
```

看到这里你可能会有疑问, `$builder->where($condition[0], $condition[1], $condition[2]);` 真的完备吗? 当然不完备. 一个非常基础的情况都无法处理---`IN`.

我注意到在Illuminate 源码中`QueryBuilder`类里有这样一处代码:

```php
    /**
    * Determine if the given operator and value combination is legal.
    *
    * Prevents using Null values with invalid operators.
    *
    * @param  string  $operator
    * @param  mixed  $value
    * @return bool
    */
protected function invalidOperatorAndValue($operator, $value)
{
    return is_null($value) && in_array($operator, $this->operators) &&
            ! in_array($operator, ['=', '<>', '!=']);
}

/**
    * Determine if the given operator is supported.
    *
    * @param  string  $operator
    * @return bool
    */
protected function invalidOperator($operator)
{
    return ! in_array(strtolower($operator), $this->operators, true) &&
            ! in_array(strtolower($operator), $this->grammar->getOperators(), true);
}

```

而对应的`$this->operators`是这样的

```php
/**
    * All of the available clause operators.
    *
    * @var array
    */
public $operators = [
    '=', '<', '>', '<=', '>=', '<>', '!=',
    'like', 'like binary', 'not like', 'between', 'ilike',
    '&', '|', '^', '<<', '>>',
    'rlike', 'regexp', 'not regexp',
    '~', '~*', '!~', '!~*', 'similar to',
    'not similar to', 'not ilike', '~~*', '!~~*',
];
```

很明显注意到, 有效的操作符里并不包含`in`, 所以这样的操作只能用` QueryBuilder::whereIn`来处理了. 所以上面的代码应该改成

```php
// rev 02
<?php
public function build(array $conditions)
{
    return function (Builder $builder) use ($conditions) {
        if (empty($conditions)) {
            return $builder;
        }

        foreach ($conditions as $condition) {
            if (strtolower($condition[1]) === 'in') {
                call_user_function_array([$builder, 'whereIn'], $condition);
            } else {
                call_user_function_array([$builder, 'where'], $condition);
            }
        }

        return $builder;
    }
} 
```

这样关于**且**这种场景就差不多覆盖了. 下面来说说**或**的情况.

对于所有给定的所有条件之前都是**或**的关系而言, 只需要把上面的`where`换成`orWhere`, 把`whereIn`换成`orWhereIn`即可. 不再赘述.

```php
// rev 03
<?php
public function buildOr(array $conditions)
{
    return function (Builder $builder) use ($conditions) {
        if (empty($conditions)) {
            return $builder;
        }

        foreach ($conditions as $condition) {
            if (strtolower($condition[1]) === 'in') {
                call_user_function_array([$builder, 'orWhereIn'], $condition);
            } else {
                call_user_function_array([$builder, 'orWhere'], $condition);
            }
        }

        return $builder;
    }
} 
```

那么关键的问题来了, 正常的查询当然不会只有且或只有或, 会两种并存, 并且二者之间是且的关系. 也就是
`where (a = 'a' and b ='b') and (c = 'c' or d = 'd')`这种.

首先明确一点, 在 Illuminate Database 中, 每出现一个`where`,最终生成的 SQL 语句就多一对`()`, 所以按照我上面的两个`builder`方法, 这种情况会生成类似`where ((a = 'a') and (b ='b')) and ((c = 'c') or (d = 'd'))`, OMG 这要怎么办?

既然或和且分属于最外层的两个括号, 那它们肯定分别在一个 where 中, 设所有 `and`条件为`$where1`, 所有`or`条件为`$where2`, 在最终的builder 中, 肯定会有这样一段`$db->table(self::TABLE)->where($where1)->where($where2)`. 为了不改变`query`方法的实现, 最直接有效的方法就是在两个条件之外再套一层闭包, 也就是这样

```php

$where1 = function (Builder $builder) use (xxxx) {
    return $builer->where(xxxx);
};

$where2 = function (Builder $builder) use (xxxx) {
    return $builer->where(xxxx);
};

$where = function (Builder $builder) use ($where1, $where2) {
    return $builder->where($where1)
    ->where($where2);
}
```

因为这种情况比较少用到, 做特殊处理是可以接受的.

写到这里, 读者可能对我设计的这种数组的写法有疑问, 为什么要写成`[['a', '=', 'd'], ['f', '=', 'b']]`这种而不是`['a' => ['=', 'd'], 'f' => ['=', 'b']]`呢? 显然后者以列名作为 key, 可以更方便的描述条件, 但问题是"或"这种情况下并列的条件中并不一定是不同的列之间, 也可能是同一列, 如果用后者来表示`a > 20 or a < 10` 该怎么做? 所以只好用了前面这种不太友善的表达方式.

更进一步思考

```php
    $where = function (Builder $builder) use ($where1, $where2) {
    return $builder->where($where1)
    ->where($where2);
}
```

这段代码和`build`方法并没有本质的区别, 所以其实它也可以被放在` build`中. 

```php
// rev 04
<?php
public function build(array $conditions)
{
    return function (Builder $builder) use ($conditions) {
        if (empty($conditions)) {
            return $builder;
        }

        foreach ($conditions as $condition) {
            if (is_closure($condition) || (is_array($condition) && in_array($condition[1], $builder->operators, true))) {
                call_user_func_array([$builder, 'where'], (array)$condition);
            } else {
                call_user_func_array([$builder, 'where' . ucfirst(strtolower($condition[1]))], (array)$condition);
            }
        }

        return $builder;
    }
}
```

这样前面提到的两个闭包的情况就可以写成

```php
$where = build([$where1, $where2]);
```

关于这个话题可讨论的还有很多, 但更复杂的情形其实可以用更底层的接口来实现, 这里就不再继续展开了.