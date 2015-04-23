---
layout: post
title: "Learning Async #2: Controlling the Context"
tags: async
---

**The main purpose of `async`/`await` is to handle the bookkeeping work when
you make asynchronous calls.**

When you do asynchronous work, there's often some context you have to be aware
of. For example, you can't touch the UI from any thread but the special "UI
thread". There are two rules about this:

1. Only the UI thread may update the UI.
2. Don't block the UI thread with long-running work.

If you have a long computation, you can spawn a thread so the UI thread isn't
blocked, but you must remember to get back onto the UI thread before you touch
any controls:

```c#
void ButtonPressed(sender s, EventArgs e)
{
    // Start some calculation on a background thread
    new Thread(new ThreadStart(() =>
    {
        var answer = Factorial(1000000);
        // Get back onto the UI thread before continuing
        Invoke(() =>
        {
            answerTextBox.Text = answer.ToString();
        });
    }));
}
```

In WinForms, the UI thread is your "context" you have to get back after doing
some asynchronous work. You use `Invoke(action)` to do that.

Before C# 5, this could be painful, but `async` and `await` make it a piece of
cake. They make that mess as simple as:

```c#
async void ButtonPressed(sender s, EventArgs e)
{
    var answer = await FactorialOnThreadPoolThreadAsync(1000000);

    // The await got us back onto the UI thread automatically
    answerTextBox.Text = answer.ToString();
}
```

(I cheated a little by assuming somebody wrote
`FactorialOnThreadPoolThreadAsync(int)` for me).

That little `await` keyword did all of this for you:

1. Save the task returned by `FactorialOnThreadPoolThreadAsync()`

2. Capture the current **"SynchronizationContext"**

3. Schedule the rest of `ButtonPressed()` to run with the current
   SynchronizationContext when the task completes. Since this is a WinForms
   app, that means it gets us back onto the thread we started on.

Notice that `await` captures a SynchronizationContext, not a thread. In a
WinForms app, these pretty much mean the same thing, but in an ASP.NET app,
they don't. In ASP.NET, the SynchronizationContext contains the _request
context_: Who is authenticated, what session state is available, etc., almost
none of which is specific to the thread.

The details of all this aren't usually important to understand. The important
thing to remember is "using `await` gets me back where I was before the
asynchronous call".
