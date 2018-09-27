# Javascript 让你的排序稳一些

最近在做一个TSlint的自定义rule，由于我们公司有太多的风格要求，这个工具是很必要的。同时该rule也会和其他rule一样，提供一个fixer，来帮助你自动修复代码。但是在开发中还是遇到了很多问题，其中有一个场景是我需要对很多的import语句进行排序，这就导致了我需要一个稳定的排序算法，因为import的先后顺序有的时候是很重要的。这时候就不能用js的sort，因为js很依赖于浏览器（嗯，node是不依赖于浏览器的，但是node的解释器还是v8），在除开firefox的其他几大浏览器在sort的实现上都用的快速排序，firefox自己用了归并排序，我们知道快速排序是一种不稳定的排序，而归并排序是一种稳定的排序，至于为什么这样，得涉及到一些历史原因，这里就不再赘述了。

要想让排序稳定，那我们得在对比两个相同的元素的时候，找到一个可以对比的戳记，这个就好办了， 我们可以用map给每个元素都加上一个戳记。那具体我们怎么去改造这个sort呢？用monkey patch就好了啊

看看具体实现

```
'use strict';

const INDEX = Symbol('index');

function getComparer(compare) {
    return function (left, right) {
        let result = compare(left, right);

        return result === 0 ? left[INDEX] - right[INDEX] : result;
    };
}

function sort(array, compare) {
    array = array.map(
        (item, index) => {
            if (typeof item === 'object') {
                item[INDEX] = index;
            }

            return item;
        }
    );

    return array.sort(getComparer(compare));
}
```

这里使用Symbol就是为了防止与现有的其他属性的键重复。
上面的例子为每个元素都赋上了INDEX
为了让排序稳定，这段代码尤其重要。对比两个元素是否相同，如果相同的话，就对比它们的INDEX的属性值，INDEX的值是它们各自的索引，所以是能够比较的

```
result === 0 ? left[INDEX] - right[INDEX] : result;
```

这就是解决排序稳定的一个简单的做法。