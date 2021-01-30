# Swift 中的 MVVM 模型

&ensp;&ensp;&ensp;&ensp;**MVVM**（Model–View–ViewModel）是一种软件架构模式，本质上是 MVC 的改进版。MVVM 确定了代码在应用程序中的位置，与应用程序中需要实时反馈的用户界面对应。它将用户界面与后端逻辑分开，抽象出 Model, View, ViewModel 3 个部分，有助于分离界面与后端逻辑的开发。这里结合斯坦福的 Swift 课程，用一个简单的 app 例子来对 MVVM 做一个简单的介绍

<img src="https://raw.githubusercontent.com/TortoiseKnightB/ios-LearningNotes/main/images/app.png" alt="app 展示图" width="500" align=center />

&ensp;&ensp;&ensp;&ensp;如图，该 app 展示了一组卡片。当点击卡片时，对应卡片翻转。

&ensp;&ensp;&ensp;&ensp;**Model** 是后台数据与逻辑的封装，独立于 View。在这个卡片 app 中 Model 就是卡片的内容与选择卡片时发生的逻辑（如当我们点击卡片时，卡片会翻转，卡片的正反面状态会发生改变）

&ensp;&ensp;&ensp;&ensp;**View** 是界面视图部分，反映了后台数据的变化（当卡片翻转时，界面视图需要显示出翻转后的卡片的样子）。VIew 本身是没有状态的，它只是展示当前 Model 的状态。当 Model 改变时，View 也需要作出相应的改变。

&ensp;&ensp;&ensp;&ensp;**ViewModel** 将 VIew 与 Model 绑定，于是当 Model 发生改变时，View 也会发生相应的改变。Model 发生的改变有时候可能会非常复杂，这时候 ViewModel 应处理并简化这些改变，再将它们传递给 View。这有助于 View 的自动更新。

&ensp;&ensp;&ensp;&ensp;MVVM 的变化涉及了两个方向，即从 Model 流向 View 与 从 View 流向 Model。

&ensp;&ensp;&ensp;&ensp;**Model => View：** 

1. Model 发生改变
2. ViewModel 会注意到 Model 的改变
3. ViewModel 解释 Model 的改变并转换成其他格式
4. ViewModel 广播该改变。（ViewModel 并不直接告诉 View 发生了什么改变，只是广播有改变发生了。负责监视这一改变的 View 会收到广播，然后重新从 ViewModel 提取数据，并重绘自己，完成相应的改变）
5. View 观察到改变，重新获取数据，重绘自己完成改变

<img src="https://raw.githubusercontent.com/TortoiseKnightB/ios-LearningNotes/main/images/model_view.png" alt="截屏2021-01-30 下午8.05.54" width="600" align=center  />

&ensp;&ensp;&ensp;&ensp;**View => Model：**（当用户点击卡片时）

1. 用户进行某些操作（点击卡片），View 向 ViewModel 发送用户的操作信息
2. ViewModel 接收并处理用户的意图（用户想要点击某一卡片）
3. ViewModel 对 Model 进行相应的改变（后台的卡片数据更改）

&ensp;&ensp;&ensp;&ensp;注意，当因为用户操作导致 Model 发生改变后，又会进行上诉的 Model => View 的过程（获取更改后的卡片数据，并重新绘制界面）

<img src="https://raw.githubusercontent.com/TortoiseKnightB/ios-LearningNotes/main/images/v_m.png" alt="截屏2021-01-30 下午8.12.52" width="600" align=center />

&ensp;&ensp;&ensp;&ensp;下面在具体的代码中介绍如何实现简单的  MVVM 模型（所有代码均来自斯坦福 Swift 课程）

```swift
//  View
//  EmojiMemoryGameView.swift
//  卡片界面部分的主要代码
struct EmojiMemoryGameView: View {
    @ObservedObject var viewModel: EmojiMemoryGame

  	// 生成卡片
    var body: some View {
        return HStack {
            ForEach(viewModel.cards) { card in
                CardView(card: card).onTapGesture {
                    self.viewModel.choose(card: card)
                }
            }
        }
                .foregroundColor(.orange)
                .padding()
    }
}


//  ViewModel
//  EmojiMemoryGame.swift
class EmojiMemoryGame: ObservableObject {
  	// Published 每次变量 model 发生变化时，进行广播
    @Published private var model: MemoryGame<String> = EmojiMemoryGame.createMemoryGame()  
    
    static func createMemoryGame() -> MemoryGame<String>{
        let emojis: Array<String> = ["👻","🎃","🕷"]
        return MemoryGame<String>(numberOfPairsOfCards: emojis.count) { pairIndex in
            return emojis[pairIndex]
        }
    }
    
    // Mark - Access to the Model
    var cards: Array<MemoryGame<String>.Card>{
        model.cards
    }
    
    // Mark: - Intent(s)
    func choose(card: MemoryGame<String>.Card) {
        model.choose(card: card)
    }
}


//  Model
//  MemoryGame.swift
//  Model 部分代码
struct MemoryGame<CardContent> {
    var cards: Array<Card>
    
    mutating func choose(card: Card){     
        print("card chosen: \(card)")
        let chosenIndex: Int = self.index(of: card)
        self.cards[chosenIndex].isFaceUp = !self.cards[chosenIndex].isFaceUp
    }
    ...
    ...
    struct Card: Identifiable {
        var isFaceUp: Bool = true
        var isMatched: Bool = false
        var content: CardContent
        var id: Int
    }
}
```

&ensp;&ensp;&ensp;&ensp;在 View 部分中，结构体 EmojiMemoryGameView 中有一个 EmojiMemoryGame 型的变量 viewModel，通过该变量建立 View 与 ViewModel 的联系。

&ensp;&ensp;&ensp;&ensp;在 ViewModel 部分中，类 EmojiMemoryGame 中有一个 MemoryGame 型的变量 model，通过该变量 ViewModel 能够获取 Model 的数据与业务逻辑。

&ensp;&ensp;&ensp;&ensp;在用户点击卡片时，ViewModel 处理用户意图，并改变后台数据 Model。ViewModel 注意到 Model 数据的改变，并将处理后的改变广播给 View。View 收到广播后重新通过 ViewModel 获取数据，并重绘界面。	

&ensp;&ensp;&ensp;&ensp;为了实现上诉过程，在 ViewModel 部分中，类 EmojiMemoryGame 需要添加协议 ObservableObject，并在类里面需要注意变化的变量 model 前加上 @Published。这样每当 Model 发生变化时，ViewModel 都会注意到该变化并进行广播。

&ensp;&ensp;&ensp;&ensp;在 View 部分中，在类 EmojiMemoryGameView 中的变量 viewModel 前加上 @ObservedObject。这样每当 ViewModel 进行广播时，View 就能观察到是否有改变，并作出相应改变。因为 Model 部分中的结构体 Card 添加了 Identifiable 协议，有唯一的 id 可识别，所以 View 在重新绘制界面的时候不会重绘所有 Card，只会重绘发生改变的 Card。

