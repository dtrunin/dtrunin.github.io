---
layout: post
title: "Graceful Shutdown in Node.js"
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

The SIGHUP (“hang-up”) signal is generated when the terminal window is closed, and under various similar conditions.
It can have a listener installed, however Node.js will be unconditionally terminated by Windows about 10 seconds later. 
On non-Windows platforms, the default behavior of SIGHUP is to terminate Node.js, 
but once a listener has been installed its default behavior will be removed.

### NestJS graceful shutdown

NestJS is a framework for building Node.js server-side applications, that uses under the hood Express HTTP server.
To shut down a NestJS application, we should explicitly call to `app.close()`, or configure the application 
to listen to termination signals, that is disabled by default.

```javascript
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    
    // Starts listening to shutdown hooks
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

### Docker

In order to implement graceful shutdown of a containerized application we should know how processes in Linux work.

#### Processes in Linux

Linux processes are ordered in a tree. The root process is called init process and has PID 1.

Any process is able to spawn a child process and get the result of its execution - exit code.
To receive an exit code of a child process the `waitpid` function is used.

The `waitpid` suspends execution of the calling process until a child specified by pid argument has finished.
When the child process finishes, `waitpid` returns its exit code.
If at the moment of calling the child has already finished, then `waitpid` returns its exit code immediately.

The `waitpid` forces blocking of the parent process.
The parent process can observe the status of a children process without blocking. 
When the child process finishes, the kernel sends to the parent process the `SIGCHLD` (sig child) signal.
The parent process can use the `SIGCHLD` signal to call the `waitpid` function to receive an exit code of the child process.

When the child process finishes, its memory and resources are deallocated, 
but a record in the process table is still exists, because the record contains the exit code of the process,
that should be obtained by the parent process.
The `waitpid` function reads exit code from this record and removes the record from the process table.
This process is called `reaping`.

A finished process, a parent of which hasn't called `waitpid`, is called `zombie`.
The only resource a zombie process uses is an entry in the process table.
Big amount of zombies can lead to inability to add new processes into the process table.

When the parent process dies, the orphaned child processes are adopted by the init PID 1 process.
This is done by the kernel.
Therefore, the init process has a special responsibility - reap adopted processes.

#### Run Node.js Application in Docker

A docker container has its own isolated process tree. A process that is run in a container has PID 1.

As we already know the PID 1 process is the init process, it should be able to reap adopted processes.
The problem is that Node.js was not designed to be the init process.
Node.js doesn't reap adopted processes, they become zombies.

There is another problem with the PID 1 process inside a container.
Such a process is [treated specially by Linux](https://docs.docker.com/engine/reference/run/#foreground):
it ignores any signal with the default action.
This means that if Node.js process has PID 1 in a container,
the process will not terminate on SIGTERM unless it is coded to do so.

These problems can be solved in the following ways

- Use the [--init](https://docs.docker.com/engine/reference/run/#specify-an-init-process) 
flag when run a container - `docker run --init CONTAINER`.
In such a case docker starts an init process as PID 1.
By default, docker uses [tini](https://github.com/krallin/tini) as an init process.
Tini spawns your executable as a child, and waits for it to exit all the while reaping zombies and performing signal forwarding. 
- Explicitly use tini or an alternative as an entry point and set your executable as a tini param.
```dockerfile
ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "./main.js"]
```

#### Graceful Shutdown of Node.js Application in Docker

Docker supports graceful shutdown of applications that are run in containers.
The `docker stop CONTAINER` command stops a running container.
The PID 1 process inside the container will receive SIGTERM, and after a grace period, SIGKILL.

As a result if a container uses an init process then graceful shutdown works out of the box.

### References / Further Reading

[Code sample of containerized NestJS application](https://github.com/dtrunin/nodejs-graceful-shutdown) <br>
[Graceful shutdown in Node.js](https://hackernoon.com/graceful-shutdown-in-nodejs-2f8f59d1c357) <br>
[Node.js signal events](https://nodejs.org/api/process.html?ref=hackernoon.com#process_signal_events) <br>
[Terminal signals](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html?ref=hackernoon.com) <br>
[Node.js process.exit([code]) docs](https://nodejs.org/api/process.html#process_process_exit_code) <br>
[Docker and the PID 1 zombie reaping problem](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/) <br>
[NestJs lifecycle events](https://docs.nestjs.com/fundamentals/lifecycle-events)
