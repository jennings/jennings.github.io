---
layout: post
title: "Learning Async #1: Basic Lessons"
tags: async
---

So I've been building [Turbocharged.Beanstalk][tb], a .NET client library for
using [Beanstalkd][beanstalkd]. This project has been a way to start using the
async/await feature of C# 5.0.

I'm writing a few articles about what I've learned from this exercise.

[tb]: https://github.com/jennings/Turbocharged.Beanstalk
[beanstalkd]: http://kr.github.io/beanstalkd/

## Tasks are not just about multithreading

A Task is not necessarily "work running on a background thread". It often is,
but not always.

Here are two method calls that block the thread:

```c#
// CPU-bound
Factorial(1000000);

// Not CPU-bound
socket.Receive(buffer, 0, 1, SocketFlags.None);
```

Both of these methods take a really long time to return. The first one takes a long
time because calculating the factorial of one million requires a lot of work.

The socket read takes a long time because the network is _soo slooowww_
compared to the CPU. But the thread isn't doing anything, it's just standing
around. Couldn't it be doing other work, something that actually requires the
CPU?

Yes. Tasks help you avoid blocking a thread when really you're just waiting for
a [system interrupt][interrupt].

[interrupt]: http://en.wikipedia.org/wiki/Interrupt


### It's like cooking

If you're the only cook in the kitchen, then you can only be doing one thing at
a time. You are a single-threaded cook.

If you're going to whip up some eggs, one of the tasks you must perform is heating
the skillet. Do you put the skillet on the stove, then stand around until it's
hot enough?

Of course not. Heating a pan doesn't require any work from you. You put the
skillet on the stove, then _while it's heating up_ you do other work:

1. Crack some eggs and scramble them
2. Get the toaster out and put the toast in
3. etc.

By the time the pan is hot, you've completed a bunch of work, none of which
depended on the pan being hot.

### Back to code

Let's read a byte from the network:

```c#
await networkStream.ReadAsync(buffer, 0, 1);
```

While this task is awaiting, _no thread is blocked waiting for the read to
complete._ Waiting for a byte to be received from the network does not require
a thread. Instead of wasting a thread until a byte is received, the thread goes
off and does useful work. Eventually, the operating system notifies your app
that the read task has completed. A thread resumes at the `await` and continues
where it left off.

Putting some work in a Task does **not** mean a background thread is blocked
waiting for the Task to complete.

The `await` isn't doing any threading magic for us. Here's another example:

```c#
public static void Main()
{
    var tcs = new TaskCompletionSource<string>();
    Task task = tcs.Task;

    int countdown = 3;
    while (!task.IsCompleted)
    {
        Console.WriteLine("{0}...", countdown);
        if (--countdown == 0)
            tcs.SetResult("Done!");
    }

    Console.WriteLine("Task completed with result: {0}", task.Result);
    Console.ReadKey();
    
    // Prints:
    //   3...
    //   2...
    //   1...
    //   Task completed with result: Done!
}
```

There's only one thread here. There are no awaits. Instead of representing code
running on a background thread, the Task returned by
`TaskCompletionSource<T>.Task` represents "waiting for somebody to call
`SetResult(T)`".
