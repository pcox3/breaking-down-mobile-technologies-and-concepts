
# Understanding The Relationship Between Lifecycle and Cocurrency In Mobile Development

Lifecycle and Concurrency are two important subject in software development and talking about mobile 
development, they influence the overall perfoermance of an app. 

### What is Lifecycle?

The lifecycle of a software component defines its journey from creation to destruction. 
**Understanding this lifecycle is crucial for managing resources, performance, and behavior**. 
While all components begin with creation and end with destruction, intermediate states like initialization,
active, suspension, or termination can exist, especially in stateful or managed environments.

For example, an Activity in Android, has 7 lifecycle stages: **onCreate**, **onStart**, **onResume**, **onStop**, **OnDestroy**,
**onRestart**, **onPause**. While in Flutter, a Stateless widget has 2 lifecycle stages: **build** and **dispose**, but the dispose stage
happens on the system level which means you don't get a lifecycle callback for it. This is because a stateless widget is 
immutable (doesn't hold a state). This means we've lifecyle stages that are exposed through a callback function and ones that are not expose. 

When you create a component, its lifecycle is activated. Understanding what tasks to perform at each stage of that lifecycle is crucial for building efficient and responsive systems. In Android, the onResume() lifecycle callback function of an Activity runs on the UI thread. This means you should avoid long-running tasks inside this function, as it could block the UI and degrade the user experience.


### What is Concurrency?

Concurrency is the concept of executing multiple tasks, done either synchronously or asynchronously. Synchronous execution means tasks are performed one at a time and in sequence; each task waits for the previous one to complete before starting. Asynchronous execution means a task can be started and then deferred, allowing the system to continue handling other tasks without waiting for the first one to finish. 

It is important to note that concurrency happens within a thread and if not properly managed, it can cause memory leaks, crashes or unexpected behaviors. 

### The blend between concurrency and lifecycle is this:

The execution of a component happens within a thread, either synchronously or asynchronously—and this activates the component’s lifecycle. For UI components, this execution typically occurs synchronously on the main thread to ensure immediate interaction and rendering. However, when concurrent tasks (asynchronous operations) are involved, there’s an important constraint: The concurrent task must not outlive the lifecycle of the component.

For example, if a component is executing a long-running task in a background thread, and that component is destroyed before the task completes, the system might attempt to update a component that no longer exists, leading to memory leaks, crashes, or inconsistent UI states.

### Implications and Best Practices
 • Use lifecycle-aware components to bind concurrency to lifecycle. This means, the concurrent task is automatically cancelled onced the component enters the destroyed lifecycle state.
 • For UI, always ensure updates happen only when the component is in a valid lifecycle state (e.g., STARTED or RESUMED).

 ### Example in Jetpack Compose. 

 ```val lifecycleOwner = LocalLifecycleOwner.current
            LaunchedEffect(lifecycleOwner) {
                lifecycleOwner.lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
                    withContext(Dispatchers.IO) {
                        println("Running...")
                    }
                }
            }```

The code snippet above, launches a concurrent task (coroutine - running on the background because it is marked as an IO task) which is tied to the lifecycle of the LaunchEffect composable. This means the task is automatically cancelled when the composable is destroyed and starts when the composable is created. This way, the coroutine does not outlive the calling composable. 

