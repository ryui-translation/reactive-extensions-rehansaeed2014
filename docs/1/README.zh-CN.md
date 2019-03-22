# Reactive Extensions (Rx) – 第一部分 – 替换 C# 事件

[原文](https://rehansaeed.com/reactive-extensions-part1-replacing-events/)，[英文](README.md)

我强烈建议大家使用 Reactive Extensions（Rx），尤其是那些从未接触过 Rx 的人。简而言之，Reactive Extensions 可以看成是「Linq to events」，如果你从未接触过 Rx，[建议先去这个网站了解一下](http://www.introtorx.com/uat/content/v1.0.10621.0/00_Foreword.html)，这是目前学习 Reactive Extensions 最佳的资源。

我花了很多时间来阅读有关 Reactive Extensions 的资料，但我发现鲜有例子来告诉我在代码中何时或何处才能用上 Rx。不过我可以明确的一点是，Reactive Extensions 是标准 C# 事件泥沼的直接替代品（标准 C# 事件从 C# 1.0 时代就已存在了），这篇文章就来简单介绍这方面的内容。

## 暴露事件

下例为 C# 事件的规范写法：

```c#
//基础 C# 事件模式
public class JetFighter
{
    public event EventHandler<JetFighterEventArgs> PlaneSpotted;

    public void SpotPlane(JetFighter jetFighter)
    {
        EventHandler<JetFighterEventArgs> eventHandler = this.PlaneSpotted;
        if (eventHandler != null)
        {
            eventHandler(this, new JetFighterEventArgs(jetfighter));
        }
    }
}
```

然后换用反应式代码：

```c#
//反应式事件模式
public class JetFighter
{
    private Subject<JetFighter> planeSpotted = new Subject<JetFighter>();

    public IObservable<JetFighter> PlaneSpotted
    {
        get { return this.planeSpotted.AsObservable(); }
    }

    public void SpotPlane(JetFighter jetFighter)
    {
        this.planeSpotted.OnNext(jetFighter);
    }
}
```

直到目前为止一切都还不难。我们用一个返回 `IObservable<T>` 的属性 (property) 代替事件 (event)，触发事件 (raise event) 就简单地变成了通过调用 Subject 类的 OnNext 方法来进行。最后，我们不会直接在 PlaneSpotted 属性中返回 `Subject<T>`，避免有人把它强制转换回 `Subject<T>` 后又触发了他们自己的事件。相反，我们可以使用 AsObservable 方法来得到一个指向它的引用（原文为 which returns a middle man）。

Reactive Extensions 还具有 **错误 (errors)** 和 **完成 (completion)** 的附加概念 (added concept)，这是 C# 事件所不具备的。这些都是可选的附加概念，不会要求你去直接替换 C# 事件，但这些概念值得你去了解，他们为事件增加了额外的能力，这些额外能力或许对你有用。

首先，第一个概念是 **错误处理 (dealing with errors)**。如果在你发现飞机的过程中出现异常，并且你想把这个非正常状况告知订阅者们，你可以这么做：

```c#
//反应式代码 - 通过 OnError 向众订阅者报告异常
public void SpotPlane(JetFighter jetFighter)
{
    try
    {
        if (string.Equals(jetFighter.Name, "UFO"))
        {
            throw new Exception("UFO Found")
        }

        this.planeSpotted.OnNext(jetFighter);
    }
    catch (Exception exception)
    {
        this.planeSpotted.OnError(exception);
    }
}
```

此处我们通过 OnError 方法将异常通知给所有事件订阅者。

那么什么是 **完成 (completion)** 的概念呢？假设你已经发现了所有飞机（spotted all the planes），随后你想通知所有订阅者，告诉他们*不会再有更多的飞机被发现了*。你可以这么做：

```c#
//反应式代码 - 通过 OnCompleted 向众订阅者报告，不会再有事件被触发
public void AllPlanesSpotted()
{
    this.planeSpotted.OnCompleted();
}
```

最后，把上面两段代码合并，结果如下：

```c#
//完整的反应式事件模式，包含完成与错误报告
public class JetFighter
{
    private Subject<JetFighter> planeSpotted = new Subject<JetFighter>();

    public IObservable<JetFighter> PlaneSpotted
    {
        get { return this.planeSpotted; }
    }

    public void AllPlanesSpotted()
    {
        this.planeSpotted.OnCompleted();
    }

    public void SpotPlane(JetFighter jetFighter)
    {
        try
        {
            if (string.Equals(jetFighter.Name, "UFO"))
            {
                throw new Exception("UFO Found")
            }

            this.planeSpotted.OnNext(jetFighter);
        }
        catch (Exception exception)
        {
            this.planeSpotted.OnError(exception);
        }
    }
}
```

## 消费事件

消费 Reactive Extensions 事件非常容易，这就是你第一眼看到 Reactive Extensions 时便能感受到的长处。下例是对标准 C# 事件订阅和取消订阅（此处经常会踩坑，使用者时常会忘记取消订阅这回事，进而可能导致内存泄漏）的代码：

```c#
//C# 事件订阅
public class BomberControl : IDisposable
{
    private JetFighter jetfighter;

    public BomberControl(JetFighter jetFighter)
    {
        jetfighter.PlaneSpotted += this.OnPlaneSpotted;
    }

    public void Dispose()
    {
        jetfighter.PlaneSpotted -= this.OnPlaneSpotted;
    }

    private void OnPlaneSpotted(object sender, JetFighterEventArgs e)
    {
        JetFighter spottedPlane = e.SpottedPlane;
    }
}
```

我不打算过于深入地介绍标准 C# 事件订阅，你可以使用 += 运算符订阅事件，或通过 -= 运算符来取消订阅。

同样的工作，反应式代码可以这样来写：

```c#
//反应式事件订阅
public class BomberControl : IDisposable
{
    private IDisposable planeSpottedSubscription;

    public BomberControl(JetFighter jetFighter)
    {
        this.planeSpottedSubscription = jetfighter.PlaneSpotted.Subscribe(this.OnPlaneSpotted);
    }

    public void Dispose()
    {
        this.planeSpottedSubscription.Dispose();
    }

    private void OnPlaneSpotted(JetFighter jetFighter)
    {
        JetFighter spottedPlane = jetfighter;
    }
}
```

这里要特别注意：首先，使用 Subscribe 方法来注册 PlaneSpotted 事件；其次，对事件的订阅被保存在 IDisposable 实现之中，因此随后只要回收它，就能对 PlaneSpotted 进行反注册。有意思的是，我们还可以对 IObservable\<T\> 的实现实例使用各类 Linq 查询，如：

```c#
//反应式 Linq 查询
jetfighter.PlaneSpotted.Where(x => string.Equals(x.Name, “Eurofighter”)).Subscribe(this.OnPlaneSpotted);
```

在上面的代码行中，我通过使用 Linq 查询到名为「Eurofighter」的飞机，然后只对它注册 OnPlaneSpotted 事件。至于其他更多 Linq 方法则超出了本文的范围，不过你可以看一下[这个网站](http://www.introtorx.com/uat/content/v1.0.10621.0/00_Foreword.html)。

## 小结

Reactive Extensions (Rx) 是一个非常强大的类库，它可以与其他类库配合叠加使用，如 Task Parallel Library（TPL）。Rx 虽然没有引进新功能，但却带来了全新的编码方式，帮助我们写出精悍优雅的代码——一如当年 Linq 那般。对于菜鸟来说，Rx 学起来有点难度，新手不知道到底哪里才能发挥它的优势。用 IObservable&lt;T&gt; 替换 C# 的基本事件就是它能发挥的优势之一。
