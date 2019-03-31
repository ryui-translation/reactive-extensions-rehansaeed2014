# Reactive Extensions (Rx) – Part 8 – Timeouts

[link](https://rehansaeed.com/reactive-extensions-rx-part-8-timeouts/), [Chinese](README.zh-CN.md)

In [part six](https://rehansaeed.com/reactive-extensions-part6-task-toobservable/) of this series of blog posts I talked about using Reactive Extensions for adding timeout logic to asynchronous tasks. Something like this:

````c#
//Task Parallel Library (TPL) and Reactive Extensions (Rx) Combined
public async Task<string> WaitForFirstResultWithTimeOut()
{
    Task<string> task = this.DownloadTheInternet();

    return await task
        .ToObservable()
        .Timeout(TimeSpan.FromMilliseconds(1000))
        .FirstAsync();

}
``

Last week I was working on a project and wanted to add a Timeout to my task but since it was an ASP.NET MVC project, I had no references to Reactive Extensions. After some thought I discovered another possible method of performing a timeout which may help in certain circumstances.

```c#
//CancellationTokenSource With Timeout Example
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

I’m using an overload on CancellationTokenSource which takes a timeout value. Then passing the CancellationToken to DownloadTheInternet. This method should be periodically checking the CancellationToken to see if it has been cancelled and if so, throw an OperationCanceledException. In this example you’d probably use HttpClient which handles this for you if you give it the CancellationToken.

The main reason why this method is better is that the task is actually being cancelled and stopped from doing any more work. In my above reactive extensions example, the task continues doing work but it’s result is just ignored.
