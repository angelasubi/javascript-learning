# 前言
Lodash 源码分析与学习

1. [chunk](#chunk)
2. [compact](#compact)
3. [difference](#difference) 依赖如下
* [slice](#slice)
* [isLength](#isLength)
* [isArrayLike](#isArrayLike)
* [isObjectLike](#isObjectLike)
* [isArrayLikeObject](#isArrayLikeObject)
* [baseGetTag](#baseGetTag)
* [getTag](#getTag)
* [isArguments](#isArguments)

## Array
### <span id="chunk">chunk</span>
---
chunk 接收两个参数 第一个参数是数组，第二个参数是每个块的长度，均分为大小的长度，如果不能均分，最后的就是剩余的元素。

```js
function chunk(array, size) {
    // 先拿 size的大小和0比较
    size = Math.max(size, 0)
    /* 新增：判断size是否为undefined */
    size = (typeof size === 'undefined') ? 0 : size
    // 判断数组是否为空，如果为空设长度为0 反之获取数组的长度 
    const length = array == null ? 0 : array.length
    // 如果数组的长度为0 或者size小于0的时候返回空数组
    if (!length || size < 1) {
        return []
    }

    //  定义index为数组的索引
    let index = 0
    // 定义resIndex为新生成数组的索引
    let resIndex = 0
    // 根据size生成一个新的数组result 向上取整，可能会有剩余元素的情况
    const result = new Array(Math.ceil(length / size))

    // 通过while循环不断从原数组里面去除指定长度的片段。
    while (index < length) {
        result[resindex++] = slice(array, index, (index += size))
    }
    /*
        上3行代码等于于如下   
        for(; index < length;) {
            result[resIndex] = array.slice(index, index + size)
            resIndex++;
            index = index + size
        }
    */
    return result
}

chunk(['a', 'b', 'c', 'd'], 2) 
// => [['a', 'b'], ['c', 'd']]
```

### <span id="slice">slice</span>
---
slice 接收三个参数，第一个是截取数组，第二个是截取开始位置，第三个是截取结束位置

```js
function slice(array, start, end) {
    // 判断传入的是否为数组如果是获取长度反之为0
    let length = array == null ? 0 : array.length
    // 如果数组长度为0， 则返回空的数组
    if (!length) {
        return []
    }

    /* 
    判断start和end是否存在，若存在，则传入这个值
    反之，start为0 end默认为数组的长度
    */
    start = start == null ? 0 : start
    end = end == undefined ? length : end

    /*
     判断start是否为负数，若是，比较start相反数与数组长度的大小
     如果大于 则为0 反之,为length+start的值
     相当于 从数组的后面开始数，开始截取的位置
    */
    if (start < 0) {
        start = -start > length ? 0 : (length + start)
    }
    /*
    如果end大于数组的长度，就赋值为数组的长度
    反之 判断是否小于0， 就等于end+end, 等于从后往前数结束的位置
    */
    end = end > length ? length : end
    if (end < 0) {
        end += length
    }
    /*
    如果start>end 则length = 0， 反之就把
    end-start然后向右无符号移动零位，然后把start向右无符号移动零位
    主要是变成无符号32位蒸熟，无论是负数或者小数，避免超出数组的界限
    */
    length = start > end ? 0 ((end - start) >>> 0)
    start >>>= 0

    // 定义数组索引
    let index = -1
    // 根据长度 创建新的数组
    const result = new Array(length)
    // 通过while循环 不断往新数组插入截取的新的片段
    while (++index < length) {
        result[index] = array[index + start]
    }
    return result
}

slice(['a', 'b', 'c', 'd'], 2)
// => ['c', 'd']
```

### <span id="compact">compact</span>
---
compact 接收一个参数 该参数是个数组， 用于创建一个新数组，包含原数组中所有的非假值元素

```js
function compact(array) {
    // 用来表示返回数组第索引
    let resIndex = 0
    const result = []
    
    // 若传递第数组为空 则返回空的数组
    if (array == null) {
        return result
    }

    // es6数组循环 将通过boolean转化为false的值除去
    for (const value of array) {
        if (value) {
            result[resIndex++] = value
        }
    }
    return result
}

compact([0, 1, false, 2, '', 3])
// => [1, 2 ,3]
```

### <span id="isLength">isLength</span>
---
isLength 接收一个参数，判断是该参数是否是有效的数组类长度

```js
const MAX_SAFE_INTEGER = 9007199254740991
/* value是否number且是否大于-1且除1是否为0且小于MAX_SAFE_INTEGER
    若为有效数组返回true反之false
*/
function isLength(value) {
    return typeof value == 'number' && value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER
}

isLength('3')
// => false
```

### <span id="isArrayLike">isArrayLike</span>
---
isArrayLike 接收一个参数,检测是否类似数组

```js
function isArrayLike(value) {
    /* value不为空 且 不是一个function 且 长度在有效范围内
        如果value是数组类型返回true 反之false
    */
    return value != null && type value !== 'function' && isLength(value.length)
}

isArrayLike([1, 2, 3])
// => true
```

### <span id="isObjectLike">isObjectLike</span>
---
isObjectLike 接收一个参数，检测是否类似object且不是null

```js
function isObject(value) {
    // 若为为object且不为null 返回true反之false
    return typeof value == 'object' && value !== null
}

isObjectLike([1, 2, 3])
// => true
```

### <span id="isArrayLikeObject">isArrayLikeObject</span>
---
isArrayLikeObject 接收一个参数，判断value是否为类数组的对象

```js
function isArrayLikeObject(value) {
    return isObjectLike(value) && isArrayLike(value)
}

isArrayLikeObject('abc')
 // => false
```

### <span id="baseGetTag">baseGetTag</span>
---
baseGetTag 接收一个参数，对toString().call()的封装，防止忽略Symbol.toStringTag属性值的 toString().call()

```js
// 定义objectProto 为对象的原型
const objectProto = Object.prototype
// 定义hasOwnProperty为检测一个对象是否含有特定的自身属性（不存在继承）
const hasOwnProperty = objectProto.hasOwnProperty
// 获取每个对象的类型(['object object'])，但需要 call 或 apply 来调用
const toString = objectProto.toString
/*
    toStringTag 用于对象的默认值描述的字符串值，使用Object.prototype.toString()
    判断Symbol 返回独一无二的自定义标签类型
 */
const symToStringTag = typeof Symbol != 'undefined' ? Symbol.toStringTag : undefined
function baseGetTag(value) {
    if (value == null) {
        // 在value 为空的情况下判断是否为 undefined 是否返回相应的值
        return value === undefined ? '[object Undefined]' : '[object null]'
    }
    // 如果 symToStringTag !== undefined && symToStringTag不存在与Object(value)相同的时候,内置的toString方法通过call的形式来调用
    if (!(symToStringTag && symToStringTag in Object(value))) {
        return toString.call(value)
    }

    // 瞅瞅 value是否存在symTostringTag，且不能在prototype找
    const isOwn = hasOwnProperty.call(value, symToStringTag)
    // 不存在 塞一个进去
    const tag = value[symToStringTag]
    // 定义一个标识
    let unmasked = false
    try {
        // 执行错误测试 
        value[symToStringTag] = undefined
        unmasked = true
    } catch (e) {}
    
    // 定义一个检测的类型
    const result = toString.call(value)
    // 如果执行错误测试成功
    if (unmasked) {
        if (isOwn) {
            // 现在的那个等于之前塞进去定义的那个
            value[symToStringTag] = tag
        } else {
            // 删除现在的这个
            delete value[symToStringTag]
        }
    }
    return result
}
```

### <span id="getTag">getTag</span>
--- 
getTag 接受一个参数 数据类型的检测

```js
/* 如下定义对应的类型 */
const dataViewTag = '[object DataView]',
      mapTag = '[object Map]',
      objectTag = '[object Object]',
      promiseTag = '[object Promise]',
      setTag = '[object Set]',
      weakMapTag = '[object WeakMap]'

const dataViewCtorString = `${DataView}`,
      mapCtorString = `${Map}`,
      promiseCtorString = `${Promise}`,
      setCtorString = `${Set}`,
      weakMapCtorString = `${WeakMap}`

    let getTag = baseGetTag
    // 判断是否为getTag()的数据类型是否等于定义的数据类型
    if ((DataView && getTag(new DataView(new ArrayBuffer(1))) != dataViewTag)||
        (getTag(new Map) != mapTag) ||
        (getTag(Promise.resolve()) != promiseTag) ||
        (getTag(new Set) != setTag) ||
        (getTag(new WeakMap) != weakMapTag)) {
            getTag = (value) => {
                // 获取参数的类型检测
                const result = baseGetTag(value)
                // 如果为'[object, Object]' 返回此对象的数组函数的引用，反之undefined
                const Ctor = result == objectTag ? value.constructor : undefined
                // 如果 Ctor 为 true 返回 `${Ctor}` 反之为空
                const ctorString = Ctor ? `${Ctor}` : ''

            if (ctorString) {
                // 如果成立，对应返回各自的值
                switch (ctorString) {
                    case dataViewCtorString: return dataViewTag
                    case mapCtorString: return mapTag
                    case promiseCtorString: return promiseTag
                    case setCtorString: return setTag
                    case weakMapCtorString: return weakMapTag
                }
            }
            return result
        }
    }
```

### <span id="isArguments">isArguments</span>
--- 
isArguments 接收一个参数 判断是否是arguments类型

```js
function isArguments(value) {
    //  是 object 且不是 null 且数据类型是'[object Arguments]'
    return typeof value == 'object' && value !== null && getTag(value) == '[object Arguments]'
}
```



