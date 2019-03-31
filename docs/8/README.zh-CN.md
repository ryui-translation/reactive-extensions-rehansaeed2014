# Reactive Extensions (Rx) – 第八部分 – 超时

[原文](https://rehansaeed.com/reactive-extensions-rx-part-8-timeouts/)，[英文](README.md)

在本系列的[第六篇](https://rehansaeed.com/reactive-extensions-part6-task-toobservable/)中，我们讲到 Rx 为异步任务添加超时逻辑，如：

````c#
// 结合 Task Parallel Library (TPL) 与 Reactive Extensions (Rx)
public async Task<string> WaitForFirstResultWithTimeOut()
{
    Task<string> task = this.DownloadTheInternet();

    return await task
        .ToObservable()
        .Timeout(TimeSpan.FromMilliseconds(1000))
        .FirstAsync();
}
``

上周在开发中，我想给我的任务添加一个超时（Timeout），但由于它是一个 ASP.NET MVC 项目，我并没有引用 Reactive Extensions。经过一番思考后，我发现在某些情况下有其它办法来执行超时。

```c#
// 带 Timeout 的 CancellationTokenSource 举例
var cancellationTokenSource = new CancellationTokenSource(TimeSpan.FromMilliseconds(1000));
try
{
    return await this.DownloadTheInternet(cancellationTokenSource.Token);
}
catch (OperationCanceledException exception)
{
    Console.WriteLine("Timed Out");
}
````

我使用了 CancellationTokenSource 带超时值的重载。然后将 CancellationToken 传递给 DownloadTheInternet。此方法会定时检查 CancellationToken 确定它是否已被取消。如果是，则抛出 OperationCanceledException。在本例中，你可能会使用 HttpClient 来处理你传入的这个 CancellationToken。

这种方法更好的原因主要是因为任务（tak）被确定地取消并停止所有工作。在上例面的反应式代码中，任务会继续，但结果会被忽略。
