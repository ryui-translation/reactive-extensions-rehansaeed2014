# Reactive Extensions (Rx) – 第四部分 – 替换计时器

[原文](https://rehansaeed.com/reactive-extensions-part4-replacing-timers/)，[英文](README.md)

Reactive Extensions (Rx) 是一个非常庞大的类库。在这一系列的博文中，我会努力说明使用 Rx 可以显著改善和/或简化实际应用程序的代码。到目前为止，我已经讨论了如何使用 Rx 替换标准 C# 事件（standard C# events）以及如何使用可观察事件（observables）来包装 C# 事件。

本文将讨论如何通过 Rx 提供一个比现有的 .NET API 更友好的 API 门面（a nicer API surface）。特别是我会提到 .NET 计时器（.NET Timers）。在 .NET Framework 中（译者注，此处是指 nfx，不是 .net core，作者文章成于 2014 年）有多个不通过的计时器，主要是 [System.Threading.Timer](https://msdn.microsoft.com/en-us/library/system.threading.timer%28v=vs.110%29.aspx) 和 [System.Timers.Timer](https://msdn.microsoft.com/en-us/library/system.threading.timer%28v=vs.110%29.aspx) 这两个。每种计时器都有优缺点，但我不打算详解孰优孰劣，你可以在[这篇对话中](https://stackoverflow.com/questions/1416803/system-timers-timer-vs-system-threading-timer)详细比较它们的异同。

## 现有的 .NET 计时器

下例简单演示如何使用 System.Timers.Timer（System.Threading.Timer 与之相似）。我们新启一个计时器，参数是 5000 毫秒，这个毫秒数用于计时器触发 Elapsed 事件；随后注册 Elapsed 事件；最后启动计时器。

```c#
//System.Threading.Timer 用例
public void StartTimer()
{
    Timer timer = new Timer(5000);
    timer.Elapsed += this.OnTimerElapsed;
    timer.Start();
}

private void OnTimerElapsed(object sender, ElapsedEventArgs e)
{
    // 做某些事
    Console.WriteLine(e.SignalTime);
    // 命令行打印
    // 11/03/2014 10:58:35
    // 11/03/2014 10:58:40
    // 11/03/2014 10:58:45
    // ...
}
```

## Reactive Extensions (Rx) 计时器

用 Reactive Extensions 来实现这个功能。此处我们使用 Interval 方法设置计时器间隔（timer interval），随后订阅 observable 即可。唯一的区别是我们无法通过计时器的 elapsed 事件获得 DateTime，而是获得一个代表计时器已触发的 elapsed 委托的次数（我觉得这更有用，因为你始终可以从这个数字中得到时间）。

```c#
//Reactive Extensions Interval 举例
public void StartTimer()
{
    Observable
        .Interval(TimeSpan.FromSeconds(5))
        .Subscribe(
            x =>
            {
                // 做某些事
                Console.WriteLine(x);
                // 命令行打印
                // 0
                // 1
                // 2
                // ...
            });
}
```

还没完。Reactive Extensions 还提供了另一种非常有用的方法。下例看上去与上例很像，我们用了 Timer 方法而不是 Interval 方法。这个方法只会调用一次计时器的 elapsed 委托。

```c#
//Reactive Extensions 计时器举例
public void StartTimerAndFireOnce()
{
    Observable
        .Timer(TimeSpan.FromSeconds(5))
        .Subscribe(
            x =>
            {
                // 做某些事
                Console.WriteLine(x);
                // 命令行打印
                // 0
            });
}
```

Timer 方法有很多重载（override）。下例中会每间隔 5 秒钟调用一次计时器的 elapsed 委托，但只在 1 分钟之后才开始。

```c#
//Reactive Extensions 延迟计时器举例
public void StartTimerInOneMinute()
{
    Observable
        .Timer(TimeSpan.FromMinutes(1), TimeSpan.FromSeconds(5))
        .Subscribe(
            x =>
            {
                // 做某些事
                Console.WriteLine(x);
                // 命令行打印
                // 0
                // 1
                // 2
                // ...
            });
}
```

最后，关于 Interval 和 Timer 方法，我们还需要知道一件事，我们可以选择将调度器接口 IScheduler 的某种实现作为最后参数传入。这样可以让计时器的 elapsed 委托运行于任何你所选择的线程上。下例中，我们在 WPF UI 线程上订阅事件，你也可以使用其他多种调度器（Scheduler），所以请看一下。

```c#
//Reactive Extensions Timer Scheduled on Dispatcher Thread
public void StartTimerOnUIThread()
{
    Observable
        .Interval(TimeSpan.FromSeconds(5), DispatcherScheduler.Current)
        .Subscribe(
            x =>
            {
                // 操作 UI
            });
}
```

## 小结

如果你还未使用过 Reactive Extensions，那么我们继续，因为本文还未展示订阅前通过 Linq 方法修改可观察事件（observable）的能力。
