---
layout: post
title: "Graceful shutdown in Node.js"
---

### What is Graceful Shutdown

Graceful shutdown means that application finishes whatever it is doing, and then exits.
The Node.js process will exit on its own if there is no additional work pending in the event loop.
As a result for graceful shutdown we just should give the Node.js process time to finish its work. 

There are some applications that have infinite event loop, e.g. web servers.
For web server shutdown gracefully means stop receiving new requests, 
finish processing active requests, clean used resources, and then shut down.

### Termination Signals

Termination signals are used to tell a process to terminate. 
There are multiple signals, and they are used for slightly different purposes.
The default action for all of these signals is to cause the process to terminate.


When the Node.js process receives a signal - signal event will be emitted.
Such an event can be handled:

```javascript
process.on('SIGINT', signal => console.log(`Received ${signal}`));

// output: 'Received SIGINT'
```


#### SIGTERM

The SIGTERM signal is a generic signal used to cause program termination.
If  this signal has a listener installed, its default behavior will be removed (Node.js will no longer exit).
It is the normal way to politely ask a program to terminate.

The shell command `kill` generates SIGTERM by default.
Not supported on Windows.


#### SIGINT

The SIGINT (“program interrupt”) signal is sent when the user types the INTR character (Ctrl+C in terminal).
If  this signal has a listener installed, its default behavior will be removed (Node.js will no longer exit).

#### SIGQUIT

The SIGQUIT signal is similar to SIGINT, except that it’s controlled by a different key—the QUIT character, 
usually Ctrl+\ - and produces a core dump when it terminates the process.

#### SIGKILL

The SIGKILL signal is used to cause immediate program termination.
Cannot have a listener installed, it will unconditionally terminate Node.js on all platforms.

#### SIGHUP

The SIGHUP (“hang-up”) signal is generated when the console window is closed, and under various similar conditions.
It can have a listener installed, however Node.js will be unconditionally terminated by Windows about 10 seconds later. 
On non-Windows platforms, the default behavior of SIGHUP is to terminate Node.js, 
but once a listener has been installed its default behavior will be removed.

### NestJS graceful shutdown

NestJS is a framework for building Node.js server-side applications, that uses under the hood Express HTTP server.
To shut down NestJS application, we should explicitly call to `app.close()`, or configure the application 
to listen to termination signals, that is disabled by default.

```javascript
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    
    // Starts listening for shutdown hooks
    // By defult app will listen to all signals declared in the ShutdownSignal enum
    app.enableShutdownHooks();
    // We can specify certain termination signals app should listen to 
    // app.enableShutdownHooks([ShutdownSignal.SIGINT, ShutdownSignal.SIGTERM]);
    
await app.listen(3000);
}
bootstrap();
```

Let's see NestJs source code to understand what happens under the hood.

```javascript
// When we call app.enableShutdownHooks(), NestJS subscribes on passed signals
// If signals are not passed, then subscribes on all signal events declared in the ShutdownSignal enum 
signals.forEach((signal: string) => process.on(signal, cleanup));

// When a signal event happens the cleanup function is executed
const cleanup = async (signal: string) => {
    // Remove listener for all termination signals, read further why we need this
    signals.forEach(sig => process.removeListener(sig, cleanup));
    // The onModuleDestroy() and beforeApplicationShutdown() hooks are called
    await this.callDestroyHook();
    await this.callBeforeShutdownHook(signal);
    
    // Stop accepting new connections to the server, finish processing active requests
    await this.dispose();
    
    // The onApplicationShutdown() hook is called
    await this.callShutdownHook(signal);
    
    // In the begining of the cleanup function, listeners for all termination signals were removed
    // Now we resend the current signal to the current process to force the default Node.js behavior for this signal -
    // if signal doesn't have a listener, then Node.js exits
    process.kill(process.pid, signal);
}
```

### Entrypoint vs cmd

containers are designed to contain a single process. 
For instance, signals that are sent to the container are sent to the process running inside the container with PID 1.
This means that the executable will not be the container’s PID 1 - and will not receive Unix signals - so your 
executable will not receive a SIGTERM from docker stop <container>.

### References / Further Reading

[Graceful shutdown in Node.js](https://hackernoon.com/graceful-shutdown-in-nodejs-2f8f59d1c357) <br>
[Node.js signal events](https://nodejs.org/api/process.html?ref=hackernoon.com#process_signal_events) <br>
[Terminal signals](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html?ref=hackernoon.com) <br>
[Node.js process.exit([code]) docs](https://nodejs.org/api/process.html#process_process_exit_code)
[Docker and the PID 1 zombie reaping problem](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/) <br>
[NestJs lifecycle events](https://docs.nestjs.com/fundamentals/lifecycle-events)
