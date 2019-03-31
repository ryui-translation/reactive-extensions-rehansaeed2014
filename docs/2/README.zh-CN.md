# Reactive Extensions (Rx) – 第二部分 – 包装 C# 事件

[原文](https://rehansaeed.com/reactive-extensions-part2-wrapping-events/)，[英文](README.md)

有时无法用 Reactive Extensions (Rx) 事件完全替代 C# 事件，这通常是由于我们实现了一个具有 C# 事件的接口，而这个接口却不为我们所拥有。

但是，通过为 C# 事件创建 IObservable&lt;T&gt; 包装器（IObservable&lt;T&gt; wrappers）却可以做到这一点（完全替代 C# 事件），甚至我们可以对事件消费者的类完全隐藏 C# 事件。这些将介绍这些技巧。

包装 C# 事件的方法取决于所使用的事件处理程序（event handler）的类型。下面将介绍三种类型的事件处理程序，以及如何用可观察事件（observable event）来包装它们。

## 包装 EventHandler C# 事件

`FromEventPattern` 方法用于包装事件（event）。注意此处我们必须为事件指定订阅（+=）和取消订阅（-=）的委托。

```c#
//包装 EventHandler C# 事件
public event EventHandler BunnyRabbitsAttack;

public IObservable<object> WhenBunnyRabbitsAttack
{
    get
    {
        return Observable
            .FromEventPattern(
                h => this.BunnyRabbitsAttack += h,
                h => this.BunnyRabbitsAttack -= h);
    }
}
```

## 包装 EventHandler&lt;T&gt; C# 事件

本例与前例大致相当，只是我们要增加对事件参数的处理。`FromEventPattern` 方法一个包含发送者（sender）和事件参数（event arguments）的 `EventPattern <T>` 对象。此处我们仅对事件参数的内容感兴趣，因此我们通过 `Select` 方法返回 BunnRabbits 属性。

```c#
//包装 EventHandler<T> C# 事件
public event EventHandler<BunnRabbitsEventArgs> BunnyRabbitsAttack;

public IObservable<BunnRabbits> WhenBunnyRabbitsAttack
{
    get
    {
        return Observable
            .FromEventPattern<BunnRabbitsEventArgs>(
                h => this.BunnyRabbitsAttack += h,
                h => this.BunnyRabbitsAttack -= h)
            .Select(x => x.EventArgs.BunnyRabbits);
    }
}
```

## 包装自定义事件处理程序来处理 C# 事件

一些 C# 事件会使用自定义事件处理程序（custom event handler），此时我们需要把事件处理程序的类型指定为 `FromEventPattern` 方法的泛型参数。

```c#
//包装自定义事件处理程序来处理 C# 事件
public event BunnRabbitsEventHandler BunnyRabbitsAttack;

public IObservable<BunnRabbits> WhenBunnyRabbitsAttack
{
    get
    {
        return Observable
            .FromEventPattern<BunnRabbitsEventHandler, BunnRabbitsEventArgs>(
                h => this.BunnyRabbitsAttack += h,
                h => this.BunnyRabbitsAttack -= h)
            .Select(x => x.EventArgs.BunnRabbits);
    }
}
```

## 通过使用显式接口实现来隐藏已有的事件

上述方法有个缺点，就是我们此刻有了两种访问事件的方法，一种是老式 C# 事件，一种是新的 Reactive Extensions 事件。通过一些特殊技巧，我们可以巧妙地在某些情况下隐藏 C# 事件。

`INotifyPropertyChanged` 接口是 XAML 开发人员们常用的接口，该接口只有一个唤作 `PropertyChanged` 的事件。我们可以通过显示实现这个接口来隐藏 PropertyChanged 这个 C# 事件（点击[此处](https://stackoverflow.com/questions/143405/c-sharp-interfaces-implicit-implementation-versus-explicit-implementation)了解隐式实现与显示实现接口的异同）。然后我们照之前那样包装事件。

现在，只有先将对象转换为 INotifyPropertyChanged 之后才能访问 PropertyChanged C# 事件（在 XAML 语言中绑定，以便此接口继续工作）。如此一来，我们新的 Reactive Extensions 可观察事件（observable event）就成了订阅属性变更事件的默认方法了。

```c#
//通过使用显式接口实现来隐藏已有的事件
public abstract class NotifyPropertyChanges : INotifyPropertyChanged
{
    event PropertyChangedEventHandler INotifyPropertyChanged.PropertyChanged
    {
        add { this.propertyChanged += value; }
        remove { this.propertyChanged -= value; }
    }

    private event PropertyChangedEventHandler propertyChanged;

    public IObservable<string> WhenPropertyChanged
    {
        get
        {
            return Observable
                .FromEventPattern<PropertyChangedEventHandler, PropertyChangedEventArgs>(
                    h => this.propertyChanged += h,
                    h => this.propertyChanged -= h)
                .Select(x => x.EventArgs.PropertyName);
        }
    }

    protected void OnPropertyChanged(string propertyName) =>
        this.propertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));

}
```

## 小结

虽然无法做到总能摆脱传统 C# 事件（legacy C# events），但我们可以把它们包装进 Reactive Extensions 可观察事件（observable event）中，甚至完全隐藏它们。
