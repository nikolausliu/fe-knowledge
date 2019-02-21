# 算法题
## 1. 数组扁平化。
```javascript
var arr = [1, 2, 3, [1, 2, 3, 4, [1, 2, 3]]]

var flatten = function (input) {
  // 取一个input的副本作为栈
  var stack = [...input]
  // 要返回的结果数组
  var res = []
  // 只要栈里还有项，就循环
  while (stack.length) {
    // pop栈里的一项
    var next = stack.pop()
    if (Array.isArray(next)) {  // 如果这一项是数组，就把数组的每一项都push进栈
      stack.push(...next)
    } else {  // 如果这一项不是数组，直接把这一项push进结果数组
      res.push(next)
    }
  }

  // 由于先push进来的项是原数组的末尾，所以reverse
  return res.reverse()
}

console.log(flatten(arr)) // [ 1, 2, 3, 1, 2, 3, 4, 1, 2, 3 ]
```