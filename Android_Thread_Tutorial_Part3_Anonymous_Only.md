# Android Thread Tutorial Part 3 (Anonymous Classes)

In this tutorial, we will dive into the concept of anonymous classes in Android and how they can be used with threads. Anonymous classes are a way to create a class without explicitly defining it, and they can be particularly useful in scenarios where you need to implement an interface or extend a class for a short period of time.

## What are Anonymous Classes?

Anonymous classes are expressions that define a subclass of a class or an implementation of an interface at the point of instantiation. They allow you to implement methods of a class or interface without creating a named class.

### Example of Anonymous Class

Here’s a simple example of how to use an anonymous class to create a thread:

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            Log.d("Thread", "Logging every second: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}).start();
```

### Using runOnUiThread

In Android, sometimes you need to update the UI from a background thread. The `runOnUiThread` method allows you to do this easily:

```java
runOnUiThread(new Runnable() {
    @Override
    public void run() {
        // Update UI elements here
    }
});
```

### Timer Example with Anonymous Thread

In this example, we will create an anonymous thread that logs a message every second for 10 seconds:

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            Log.d("Timer", "Logging every second: " + (i + 1));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}).start();
```

Now you know how to use anonymous classes with threads in Android! Keep coding and exploring!