
方法链式调用

Method chaining is the practice of object methods returning the object itself in order for the result to be called for another method. Like this:

方法链式调用的意思：通过对想象方法返回的对象作为结果，用于调用另一个方法, 如下：

```participant.addSchedule(events[1]).addSchedule(events[2]).setStatus('attending').save()```

This seems to be considered a good practice, since it produces readable code, or a "fluent interface". However, to me it instead seems to break the object calling notation implied by the object orientation itself - the resulting code does not represent performing actions to the result of a previous method, which is how object oriented code is generally expected to work:

当这种写法能够产生一些“可读的”代码，“通顺的”交互的时候，这看起来应该被认为是一个好的做法，但是对于我来说，这种写法已经打破了面向对象编程的思想——对象“调用”方法。这里的每处的结果实际上并**不表示**前一个方法所正在执行的操作的结果，但是面向对象编程的思想，正是希望每处的结果为该对象所正在执行操作的结果

```participant.getSchedule('monday').saveTo('monnday.file')```
This difference manages to create two different meanings for the dot-notation of "calling the resulting object": In the context of chaining, the above example would read as saving the participant object, even though the example is in fact intended to save the schedule object received by getSchedule.

这里不同的管理者为“点符号”创建了两个不同的意义 

I understand that the difference here is whether the called method should be expected to return something or not (in which case it would return the called object itself for chaining). But these two cases are not distinguishable from the notation itself, only from the semantics of the methods being called. When method chaining is not used, I can always know that a method call operates on something related to the result of the previous call - with chaining, this assumption breaks, and I have to semantically process the whole chain to understand what the actual object being called really is. For example:

```participant.attend(event).setNotifications('silent').getSocialStream('twitter').postStatus('Joining '+event.name).follow(event.getSocialId('twitter'))```
There the last two method calls refer to the result of getSocialStream, whereas the ones before refer to the participant. Maybe it's bad practice to actually write chains where the context changes (is it?), but even then you'll have to constantly check whether dot-chains that look similar are in fact keep within the same context, or only work on the result.

To me it appears that while method chaining superficially does produce readable code, overloading the meaning of the dot-notation only results in more confusion. As I don't consider myself a programming guru, I assume the fault is mine. So: What am I missing? Do I understand method chaining somehow wrong? Are there some cases where method chaining is especially good, or some where it's especially bad?

Sidenote: I understand this question could be read as a statement of opinion masked as a question. It, however, isn't - I genuinely want to understand why chaining is considered good practice, and where do I go wrong in thinking it breaks the inherent object-oriented notation.
