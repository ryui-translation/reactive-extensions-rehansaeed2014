# Reactive Extensions (Rx) – 第三部分 – 命名约定

[原文](https://rehansaeed.com/reactive-extensions-part3-naming-conventions/)，[英文](README.md)

除了要用英语来表示某事已发生外（如 `PropertyChanged`），标准 C# 事件（standard C# events）并没有什么命名约定（naming convention）。那么如果属性返回一个 IObservable&lt;T&gt; 对象，我们需要命名约定么？我不能确定是否需要，但我会解释我为什么要命名约定。

C# 事件在类中很容易与属性（properties）和方法（methods）区分开，因为它们在 Visual Studio 的智能感知（Intelli-Sense）中是不同的图标。但 Visual Studio 不会给 IObservable&lt;T&gt; 属性和别的属性作任何差别。如果微软公司将来打算把 Reactive Extensions (Rx) 深度集成进 Visual Studio 的话，或许这一问题会有所改观。

使用命名约定的第二个理由是我经常使用 Reactive Extensions 事件来包装现有的 C# 事件。C# 事件和 IObservable&lt;T&gt; 属性不可能具有相同的名称。

如果你看过我前几篇文章，你就会发现我在属性名称前冠了个「When」这个词。我觉得这可以很好地表明事件已经发生，并且还可以在智能感知（Intelli-Sense）中把 Reactive Extensions 事件属性聚在一起显示。

```c#
//命名约定举例
public IObservable<string> WhenPropertyChanged
{
    get { ... };
}
```

我在一些地方看到有人给出所谓「冷可观察事件」和「热可观察事件」的建议（参见[这里](https://stackoverflow.com/questions/2521277/what-are-the-hot-and-cold-observables)的解释），说这两类可观察事件应该有不同的命名约定。我个人觉得这只是一个实现细节（implementation detail），我不名为为何一个事件的订阅者需要知道这个事件是「热的」或是「冷的」。此外，把这个概念在其他开发人员间普及意味着要不断地一遍又一遍地解释它的意义；而使用「When」是多么简单易懂、清晰明了。

这是个不错的开发性问题，你怎么看？
