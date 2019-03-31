# Reactive Extensions (Rx) – 第六部分 – 将任务转为可观察对象

[原文](https://rehansaeed.com/reactive-extensions-part6-task-toobservable/)，[英文](README.md)

## 快速回顾

在本系列前几篇文章中，我简单介绍了在实践中可以使用的 Reactive Extensions 的知识点。与旧的 .NET 代码相比，Rx 提供了更为先进整洁的接口。即，替换 C# 事件、包装现有的 C# 事件、以及替换 System.Threading.Timers（或其他 .NET 中的计时器类型）。

当你得到一个可观察对象后，你可以对它做很多事情。本系列的最后一篇文章将介绍如何等待（await）一个可观察对象。

本文则介绍另一条路径，通过将任务（task）转换为一个可观察对象（observable），并告诉你这么做的理由。

## 将任务转为可观察对象

扩展方法 `ToObservable` 使你可以将普通任务（Task）或泛型任务（Task&lt;T&gt;）转换为一个可观察对象（IObservable&lt;T&gt;）。在任务（Task）上调用 ToObservable 将返回一个 IObservable&lt;Unit&gt; 对象。Unit 是一种无任何效果的空对象，之所以要返回这么个对象……是因为压根就没有 IObservable（不是泛型的，不带 `T`）接口嘛……

```c#
//ToObservable 举例
IObservable<Unit> observable = Task.Run(() => Console.WriteLine("Working")).ToObservable();

IObservable<string> observableT = Task<string>.Run(() => "Working").ToObservable();
```

如果你订阅了上例中的可观察对象，它们将只返回一个值，然后完成（complete）。

## 结合起来

那我们何时使用这个功能？

行，我们来看几个例子，看看它们会发生点什么。假设我们有以下代码：

```c#
//Some Start-up Code
public Task<string> GetHelloString()
{
    return Task.Run(
        async () =>
        {
            await Task.Delay(500);
            return "Hello";
        });
}

public Task<string> GetWorldString()
{
    return Task.Run(
        async () =>
        {
            await Task.Delay(1000);
            return "World";
        });
}
```

当我们同时调用这些方法并期待获得第一个结果时会发生什么？与 Reactive Extensions (Rx) 相比，这段代码如何用任务并行库（TPL）来实现？

```c#
//Task Parallel Library (TPL) 举例
public async Task<string> WaitForFirstResultAndReturn()
{
    Task<string> task1 = this.GetHelloString();
    Task<string> task2 = this.GetWorldString();
    return await Task.WhenAny(task1, task2);
}
```

```c#
//Reactive Extensions (Rx) 举例
public async Task<string> WaitForFirstResultAndReturn()
{
    IObservable<string> observable1 = this.GetHelloString().ToObservable();
    IObservable<string> observable2 = this.GetWorldString().ToObservable();
    return await observable1.Merge(observable2).FirstAsync();
}
```

在 TPL 的例子中，我使用 WhenAny 方法等待第一个任务完成，然后返回结果。

在 Rx 的例子中，我把任务都转换成了可观察对象，并使用 Merge 方法把这两个可观察对象合并为一个，最后用 FirstAsync 方法等待第一个结果（我们在上一篇文章中已经介绍过如何等待一个可观察对象了）。

总的来讲，这两种技术看起来很相似，TPL 在性能上略占上风。

至于另一个例子，接下来我们将尝试等待两个结果，并将它们组合起来以获得一些有意义的结果。

```c#
//Task Parallel Library (TPL) 举例
public async Task<string> WaitForAllResultsAndReturnCombinedResult()
{
    Task<string> task1 = this.GetHelloString();
    Task<string> task2 = this.GetWorldString();
    return string.Join(" ", await Task.WhenAll(task1, task2));

}
```

```c#
//Reactive Extensions (Rx) 举例
public async Task<string> WaitForAllResultsAndReturnCombinedResult()
{
    IObservable<string> observable1 = this.GetHelloString().ToObservable();
    IObservable<string> observable2 = this.GetWorldString().ToObservable();
    return await observable1.Zip(observable2, (x1, x2) => string.Join(" ", x1, x2));
}
```

在 TPL 的例子中，我们用 WhenAll 方法等待两个任务返回的字符串数组，然后用 Join 方法连接它们并返回。

在 Rx 的例子中，我将任务转换为可观察对象后，使用 Zip 方法组合自两可观察对象中返回的结果，方法是它提供一个连接两个字符串的委托。

同样地，这两个栗子看起来也很相似，但单纯的 TPL 例子更易于理解。

再举一例，这一次我们将返回第一个结果，但在其中添加一个超时（timeout）。

```c#
//Task Parallel Library (TPL) 举例
public async Task<string> WaitForFirstResultAndReturnResultWithTimeOut()
{
    Task<string> task1 = this.GetHelloString();
    Task<string> task2 = this.GetWorldString();
    Task timeoutTask = Task.Delay(100);

    Task completedTask = await Task.WhenAny(task1, task2, timeoutTask);
    if (completedTask == timeoutTask)
    {
        throw new TimeoutException("The operation has timed out");
    }

    return ((Task<string>)completedTask).Result;

}
```

```c#
//Reactive Extensions (Rx) 举例
public async Task<string> WaitForFirstResultAndReturnResultWithTimeOut()
{
    IObservable<string> observable1 = this.GetHelloString().ToObservable();
    IObservable<string> observable2 = this.GetWorldString().ToObservable();
    return await observable1.Merge(observable2).Timeout(TimeSpan.FromMilliseconds(100)).FirstAsync();
}
```

在 TPL 的例子中，第三个任务表示超时。如果这个任务先于另两个任务完成，将抛出 TimeoutException 超时异常。

在 Rx 的例子中，我们再次合并两个可观察对象，但这次使用 Timeout 方法来实现超时的功能。

这个例子中很明显，Rx 的代码更简洁、更易于理解。

当我们把两者结合起来会发生什么？

```c#
// 组合 Task Parallel Library (TPL) 与 Reactive Extensions (Rx)
public async Task<string> WaitForFirstResultAndReturnResultWithTimeOut2()
{
    Task<string> task1 = this.GetHelloString();
    Task<string> task2 = this.GetWorldString();

    return await Task
        .WhenAny(task1, task2)
        .ToObservable()
        .Timeout(TimeSpan.FromMilliseconds(1000))
        .FirstAsync();
}
```

在这个例子中，我们在最后才使用 ToObservable 方法和 Timeout 方法。如你所见，这种组合的方法同时具备了 TPL 和 Rx 的优点，并使代码变得更易阅读。

## 小结

把任务（Task）转换为可观察对象（Observables）的第一个明确理由是可以使用 Timeout 方法。可能还会有其他理由，但我一时半会儿还想不出来。实际上我觉得我也暂时憋不出其他关于 Rx 的话题了……我通过写这个系列已经学到了很多，我希望你也是。
