---
title: How Beanstalk handles multiple workers
date: 2018-02-18T11:00:00-0800
layout: post
---

[Beanstalk][beanstalkd] is a neat little job queue. I stumbled across it years
ago and wanted to know how it worked, so I wrote a little C# client library
called [Turbocharged.Beanstalk][turbochargedbeanstalk] to teach myself. I also
learned a lot about socket programming.

[beanstalkd]: https://kr.github.io/beanstalkd/
[turbochargedbeanstalk]: https://github.com/jennings/turbocharged.beanstalk

The hardest concept for me to understand was how Beanstalk handles a worker
that works on multiple jobs on the same TCP connection. At first, it looks like
Beanstalk doesn't support this. [NSQ][nsq] has a [max_in_flight setting][rdy],
but Beanstalk has no equivalent.

Beanstalk's `DEADLINE_SOON` message is what allows clients to reserve multiple
jobs on a connection without breaking.

[nsq]: http://nsq.io/
[rdy]: http://nsq.io/clients/building_client_libraries.html#rdy_state

Someone asked me over email whether Beanstalk supports multiple workers, so I
thought I'd write it down and put in public in case it helps anyone else.


## Summary

Beanstalk supports reserving multiple jobs on a single TCP connection. Because
a pending `reserve` blocks the connection until a job arrives, the Beanstalk
server will abort any in-flight `reserve` commands with `DEADLINE_SOON` if a
reserved job is about to expire but hasn't been deleted yet.

This unblocks the connection and Beanstalk can process any `delete` commands
that were waiting to be processed.


## Definitions

- **producer** - A client connected to Beanstalk that puts jobs in a tube.

- **worker** - A client connected to Beanstalk that pulls jobs out of a tube
  and processes them.

- **job** - A job is just a sequence of bytes. It might be binary data, or it
  might be text or JSON in UTF-8. Beanstalk doesn't care, it just sees bytes.

- **tube** - A named container where jobs are stored in Beanstalk. Producers
  put jobs in a tube and workers watch tubes that contain jobs they can
  process.


## Workers

A client connects to Beanstalk and sends _commands_.

Beanstalk responds to a client's commands in order. **A client can send
multiple commands at once, but Beanstalk will not respond to subsequent
commands until it has responded to previous ones.**

When a client connects and wants to be a worker, it issues these two commands:

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
watch tweets
                            WATCHING 1
reserve
```

This begins watching the tube named "tweets". This connection is now blocked
until the server can respond to the `reserve` command.

Eventually, there will be a job ready in the tube. Beanstalk responds to the
`reserve` command with this:

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
reserve
                            RESERVED 1 21
                            this is my cool tweet
```

This job has ID #1 and is 21 bytes long. The job itself is "this is my cool
tweet".

When the client processes this job (by posting it to Twitter), it sends the
following command:

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
delete 1
                            DELETED 1
```

This deletes job ID #1. (The client could instead `bury` or `release` the job,
but those commands aren't important for this topic.)

Now the connection is idle, so the worker will probably send another `reserve`
command. The worker issues these `reserve`/`delete` commands forever to process
jobs as they come in.


## Reserving multiple jobs

If a worker can process multiple jobs concurrently, it can send the `reserve`
command multiple times.

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
reserve
reserve
                            RESERVED 1 21
                            this is my cool tweet
```

The client has reserved one job, but there are no other jobs in the tube, so
the second `reserve` command has not been answered yet.

The client finishes processing job #1 and sends a `delete` command:

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
delete 1
```

But the server won't process this `delete` because it hasn't responded to the
previous `reserve` command yet (because the tube is empty).

This looks like a bad situation: The server won't process the `delete` until it
responds to the `reserve`, and it won't do that until a new job is put in the
tube.

But maybe a new job will _never_ be put in the tube. How does Beanstalk ensure
job #1 gets deleted?


## Deadlines

If a worker gets stuck processing something, you don't want that job to be lost
forever. So all jobs have a **time to run**. If a worker reserves a job and
doesn't handle it within the time to run, Beanstalk assumes the worker has
failed and redelivers the job to another worker.

**If a worker has a reserved a job that reaches 1 second left in its time to
run, any pending `reserve` commands will abort with the message
`DEADLINE_SOON`.**. This is the magic that fixes the previous problem.

The complete exchange looks like this, assuming a 30 second time to run:

```
CLIENT COMMAND              SERVER RESPONSE
-------------------------------------------------
reserve
reserve
                            RESERVED 1 21
                            this is my cool tweet
delete 1

         ...29 seconds go by...

                            DEADLINE_SOON
                            DELETED 1
```

The `DEADLINE_SOON` is the response to the _second_ `reserve` command. This
unblocks the connection, and now Beanstalk immediately processes the `delete`
command.

**`DEADLINE_SOON` doesn't indicate any sort of failure, it's just there to unblock the
connection so any pending `delete` commands can go through.**
