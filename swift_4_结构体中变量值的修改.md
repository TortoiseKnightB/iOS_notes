# swift 结构体的一些注意事项

Swift 中函数中定义的参数一般默认是参量 let 类型，如果需要修改参数的值，需要添加关键字`inout`，如：
```swift
func changeInt(_ num: inout Int) {
    num += 1
}

var num: Int = 1
changeInt(&num)
print(num)		// 2
```

Swift 的结构体中可以定义函数，如果该函数实现了对自身的修改，那么需要在函数定义语句前加上 `mutating`关键字
```swift
struct Stu {
    var name: String
    mutating func changeName(_ newName: String) {
        self.name = newName
    }
}

var stu1: Stu = Stu(name: "张三", isInSchool: false)
print(stu1.name)
stu1.changeName("李四")
print(stu1.name)

// 张三
// 李四
```
下面再来谈论一下深拷贝与浅拷贝

在 Swift 中，struct 类型是深拷贝，将原结构体中所有变量都复制一份
```swift
var stu1: Stu = Stu(name: "张三")
var stu2 = stu1
print(stu1.name)
print(stu2.name)
stu1.name = "李四"
print(stu1.name)
print(stu2.name)
// 张三
// 张三
// 李四
// 张三
// stu1 的改变不影响 stu2
```

class 类型是浅拷贝，只拷贝了原对象的内存地址，两个对象指向同一个内容
```swift
class Book {
    var name: String
    init(_ name: String) {
        self.name = name
    }
}

var book1: Book = Book("三国演义")
var book2 = book1
print(book1.name)
print(book2.name)
book1.name = "西游记"
print(book1.name)
print(book2.name)
// 三国演义
// 三国演义
// 西游记
// 西游记
// book1 的改变引起了	book2 的改变
```

