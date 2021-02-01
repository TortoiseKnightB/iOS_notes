# swift数组技巧

#### 如何从数组中截取某一片段形成新的数组

&ensp;&ensp;&ensp;&ensp;Swift 中为数组提供了 prefix 与 suffix 函数，截取数组头部与尾部指定长度的内容，但是仍然保留了原来数组的下标

```swift
func prefix(upTo end: Int) -> ArraySlice<Int>
func suffix(from start: Int) -> ArraySlice<Int>

var arr = [1,2,3,4,5]
let preArr = arr.prefix(upTo: 2)
print(preArr)			// [1, 2]
print(preArr[0])	// 1
print(preArr[1])	// 2
let sufArr = arr.suffix(from: 3)	
print(sufArr)			// [4, 5]
print(sufArr[3])	// 4
```

&ensp;&ensp;&ensp;&ensp;如果想要把截取的数组当做新的数组来使用，则需要更新数组下标，用如下方法即可

```swift
let newArr:[Int] = Array(arr.suffix(from: 3))
print(newArr)				// [4, 5]
print(newArr[0])		// 4
print(newArr[1])		// 5
```

&ensp;&ensp;&ensp;&ensp;这样截取的数组就可以当成新的数组来使用