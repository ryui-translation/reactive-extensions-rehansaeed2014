# Reactive Extensions (Rx) – Part 3 – Naming Conventions

[link](https://rehansaeed.com/reactive-extensions-part3-naming-conventions/), [Chinese](README.zh-CN.md)

Standard C# events do not have any real naming convention, except using the English language to suggest that something has happened e.g. ‘PropertyChanged’. Should a property returning an IObservable&lt;T&gt; have a naming convention? I’m not entirely certain but I’ll explain why I have used one and why.

C# events are easily differentiated in a class from properties and methods because they have a different icon in the Visual Studio Intelli-Sense. Visual Studio does not provide IObservable&lt;T&gt; properties any differentiation. This may change in the future if Microsoft decides to integrate Reactive Extensions (Rx) more deeply into Visual Studio.

The second reason for using a naming convention is that I often wrap existing C# events with a Reactive Extensions event. It’s not possible to have the same name for a C# event and an IObservable&lt;T&gt; property.

You will have noticed already if you’ve looked at my previous posts that I use the word ‘When’ prefixed before the name of the property. I believe, this nicely indicates that an event has occurred and also groups all our Reactive Extension event properties together under Intelli-Sense.

```c#
//Naming Convention Example
public IObservable<string> WhenPropertyChanged
{
    get { ... };
}
```

I have read in a few places people suggesting that so called ‘Hot’ and ‘Cold’ (See [here](https://stackoverflow.com/questions/2521277/what-are-the-hot-and-cold-observables) for an explanation) observables should have different naming conventions. I personally feel that this is an implementation detail and I can’t see why the subscriber to an event would need to know that an event was ‘Hot’ or ‘Cold’ (Prove me wrong). Also, trying to teach this concept to other developers and get them to implement it would mean constantly looking up the meanings (I keep forgetting myself), whereas using ‘When’ is a nice simple concept which anyone can understand.

This is a pretty open question at the moment. What are your thoughts on the subject?
