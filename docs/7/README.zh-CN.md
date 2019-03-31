# Reactive Extensions (Rx) – 第七部分 – Sample 事件

[原文](https://rehansaeed.com/reactive-extensions-part7-sample-events/)，[英文](README.md)

从上次发帖介绍 Rx 至今已过去很久。这个系列的文章很受欢迎，感谢社区的积极反馈。我昨天和一个作 WPF 开发的同事聊了一下，他在 WPF 中使用了标准 C# 事件。他订阅了 TextChanged 事件，只要用户输入文本字符，就会动态更新到用户界面。他触发了太多的事件，以至于他的用户界面根本没法完成这个功能需求。

```c#
//WPF TextChanged 事件导致问题
this.TextBox.TextChanged += this.OnTextBoxTextChanged;

private void OnTextBoxTextChanged(object sender, TextChangedEventArgs e)
{
    // 重度的用户界面更新可能会导致应用程序被锁定.
}
```

这种情况非常常见，我自己也遇到过好多次。这个问题的解决方案是获得正在触发的事件样本，然后每隔几秒钟才更新一次用户界面（UI）。不用 Rx 的话这也是可行的，但你需要编写相当数量的样板代码（boilerplate code）（当然，我做到了，我骄傲）。

Reactive Extensions (Rx) 可以通过一些易于理解（这切中了用户痛点）的代码来实现这一点。第一步是包装 WPF 的 TextChanged 事件（我在先前的[帖子](https://rehansaeed.com/reactive-extensions-part2-wrapping-events/)中介绍过如何包装事件）。

```c#
//Reactive Extensions (Rx) 样本方法
public IObservable<TextChangedEventArgs> WhenTextChanged
{
    get
    {
        return Observable
            .FromEventPattern<TextChangedEventHandler, TextChangedEventArgs>(
                h => this.TextBox.TextChanged += h,
                h => this.TextBox.TextChanged -= h)
            .Select(x => x.EventArgs);
    }
}

this.WhenTextChanged
    .Sample(TimeSpan.FromSeconds(3))
    .Subscribe(x => Debug.WriteLine(DateTime.Now + " Text Changed"));
```

最后也是最简洁的步骤是使用 Sample 方法每 3 秒获得一次最新的文本变更事件，并将之传递给 Subscribe 委托。这真的很容易，莫怪本帖太短太水！
