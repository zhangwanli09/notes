# 如何实现数组随机排序

```javascript
// 随机数排序
function random1(arr) {
  return arr.sort(() => Math.random() - .5)
}

// 随机插入排序
function random2(arr) {
  const cArr = [...arr]
  const newArr = []
  while (cArr.length) {
    const index = Math.floor(Math.random() * cArr.length)
    newArr.push(cArr[index])
    cArr.splice(index, 1)
  }
  return newArr
}

// 洗牌算法，随机交换排序。（最快）
function random3(arr) {
  const l = arr.length
  for (let i = 0; i < l; i++) {
    const index = Math.floor(Math.random() * (l - i)) + i
    const temp = arr[index]
    arr[index] = arr[i]
    arr[i] = temp
  }
  return arr
}
```

洗牌原理，该算法需要遍历整个数组，当遍历到第 i（i 为数组元素的索引）个元素时，从 0 到 i 随机挑选出一个数字，记为 index，然后对索引为 i 和 index 的数组元素进行互换，直至遍历结束。如此下来，也即完成了数组的随机排序。

> [sort()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
>
> [Math.random()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math/random)
