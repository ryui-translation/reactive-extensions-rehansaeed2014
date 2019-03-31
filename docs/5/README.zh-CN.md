# Reactive Extensions (Rx) – 第五部分 – 等待可观察对象

[原文](https://rehansaeed.com/reactive-extensions-part5-awaiting-observables/)，[英文](README.md)

我刚和我的同事聊起关于 TaskCompletionSource 的各种优点，我觉得我把这个话题分享出来会很有意思，所以最后我写成了这篇关于 Reactive Extensions (Rx) 的文章。

## TaskCompletionSource

[TaskCompletionSource](https://msdn.microsoft.com/en-us/library/dd449174%28v=vs.110%29.aspx) 是个很棒的工具，它可以把任何不遵循 TPL（Task Parallel Library）模式的异步操作（asynchronous operation）转换为一个 Task。我们从下例开始讲起。

```c#
//TaskCompletionSource 用例
public Task<bool?> ShowDialog()
{
    TaskCompletionSource<bool?> taskCompletionSource = new TaskCompletionSource<bool?>();

    Window window = new MyDialogWindow();
    EventHandler eventHandler = null;
    eventHandler =
        (sender, e) =>
        {
            window.Closed -= eventHandler;
            taskCompletionSource.SetResult(window.DialogResult);
        };
    window.Closed += eventHandler;
    window.Show();

    return taskCompletionSource.Task;
}
```

上例中，我们创建一个新窗体，注册其 Closed 事件并显示该窗体。当窗体关闭时，从关闭事件处理程序（closed event handler）中进行反注册（de-register）（这是为了避免对窗体有引用残留进而捯饬内存泄漏），并使用 SetResult 方法窗体的 DialogResult 窗体给 TaskCompletionSource。

TaskCompletionSource 给我们提供了一个 Task 兑现，以便我们能够在方法结束时返回。当我们返回 Task 时，它的状态是 `WaitingForActivation`。只有当窗体关闭时才会调用 SetResult 方法，随后 Task 的状态才会变成 RanToCompletion。

在 TaskCompletionSource 的作用下，整个操作被封装起来，并美美哒地通过 `Task<bool?>` 返回。现在我们可以调用方法并等待方法调用的结果，从而使我们能够体验到 TPL 带来的功能与简便性。

```c#
//用例
bool? result = await ShowDialog();
```

当然 TaskCompletionSource 还有其他玩法。其它[更多关于 TaskCompletionSource 的信息](https://blogs.msdn.com/b/pfxteam/archive/2009/06/02/9685804.aspx)，我建议阅读 [Stephen Toub 的博客](https://blogs.msdn.com/b/pfxteam/)。

## 等待可观察对象

在向我同事展示了上述例子后，我嘚瑟地发现如果用 Reactive Extensions (Rx) 的话可以让代码变得更加简单。随着 Rx 新版本的出现，通过等待可观察对象（await observables），我们可以改写上面的例子：

```c#
//Reactive Extensions (Rx) 举例
public async Task<bool?> ShowDialog()
{
    var window = new MyDialogWindow();
    var closed = Observable
        .FromEventPattern<EventHandler, EventArgs>(
            h => window.Closed += h,
            h => window.Closed -= h);

    window.Show();
    await closed.FirstAsync()

    return window.DialogResult;

}
```

`await` 关键字是 C# 仅有的语法糖（译者注：文章成于 2014 年，因此此处这么讲也没错），它可使你在编写棘手的异步代码时游刃有余。原因就在 GetAwaiter 这个方法上。 Reactive Extensions (Rx) 团队充分发挥了 TPL 的高明之处。他们将这个方法（实际上是一个扩展方法）添加到 IObservable&lt;T&gt;，使得我们可以等待这个可观察对象。

但我还是要声明一下注意事项。在上例中，如果窗体被多次开关，Closed 事件会被多次触发（be fired any number of times），而且 Closed 事件周围的可观察对象包装器永远不会完成。因为我们的可观察对象会返回多个结果，但任务（task）只能返回单个任务。

示例代码中我们的秘诀是 `FirstAsync` 这个方法。实际上我们只需要等待可观察对象返回的第一个结果，其它结果我们根本不关心。默认情况下，如果没有这个 `FirstAsync` 方法的情况下等待一个可观察对象，它会等待这个可观察对象实际完成**之前的最后一个结果**。如果你的可观察对象没有完成，你就会**一直等下去**！

在你使用 `await` 等待结果之前，你可以使用 Reactive Extensions (Rx) 团队添加的几个实用方法。所有这些方法都以「Async」结尾。我在下面列举了这些方法（它们有很多重载，我仅列出了主要的方法）：

```c#
//Reactive Extensions (Rx) 异步扩展方法
// 返回可观察序列的第一个元素
string result = await observable.FirstAsync();

// 返回可观察序列的第一个元素，若不存在则返回默认值
string result = await observable.FirstOrDefaultAsync();

// 返回可观察序列的最后一个元素，此为等待一个可观察事件时的默认行为
string result = await observable.LastAsync();

// 返回可观察序列的最后一个元素，若不存在则返回默认值
string result = await observable.LastOrDefaultAsync();

// 返回可观察序列的唯一一个元素。如果序列中不存在元素或存在超过一个元素，则抛出异常。
string result = await observable.SingleAsync();

// 返回可观察序列的唯一一个元素，如果序列为空则返回默认值；如果序列中存在超过一个元素，则抛出异常。
string result = await observable.SingleOrDefaultAsync();

// 为可观察序列的每个元素调用一个 action，并返回序列终止时发出信号的 Task
await observable.ForEachAsync(x => Console.WriteLine(x));
```

上述所有方法都允许你从可观察对象中得到一个单一的结果（a single result）。`ForEachAsync` 略有不同，它会对每个项目（item）进行操作，当且仅当可观察对象完成时，任务才会完成。

## 小结

本文我们学习了如何以不同的方式等待可观察对象，以及它是如何以更为简洁优雅的方式来完成与 TaskCompletionSource 相同的工作。

我们还了解了在等待可观察对象时需要注意的一些事项，即可观察对象会返回多个结果，你需要选择其一作为任务（task）返回。
