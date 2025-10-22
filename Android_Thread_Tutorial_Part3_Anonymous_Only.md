# Android Thread Tutorial — Part 3 (Anonymous Classes) + Part 4 (Updating UI with runOnUiThread)

This document teaches anonymous classes in Java (for Android beginners), then shows how to implement the timer example using an anonymous thread (logging to Logcat every second), and finally explains how to update the UI from a background thread using `runOnUiThread`.

Overview:
- Part 3 — Anonymous classes: deep theory, syntax, scoping rules, common patterns, pitfalls.
- Timer example — create the timer as an anonymous Thread/Runnable and log every second (10s).
- Part 4 — Why background threads cannot update UI directly; `runOnUiThread` explained with examples.

Important note for students:
- This file focuses on anonymous classes and `runOnUiThread`. We intentionally postpone advanced thread-safety topics (like `volatile`, synchronization, interrupts, and lifecycle cleanup) to a later "Best Practices" lesson so you can master basic concepts first.

---

Part 3 — Anonymous classes (in-depth)
====================================

What is an anonymous class?
- An anonymous class is a one-off class declared and instantiated at the same time, without a name.
- It's usually used to implement an interface or extend a class when you only need a short, single-use implementation.
- In Android, anonymous classes are everywhere: `OnClickListener`, `Runnable`, `Comparator`, and tiny adapter implementations.

Why use anonymous classes?
- Reduce extra files and boilerplate for small handlers/callbacks.
- Keep implementation close to where it is used — easier to read in small examples.
- Good for short-lived behavior like event handlers or small background tasks.

Basic syntax
- Implementing an interface inline:
```java
someMethod(new SomeInterface() {
    @Override
    public void method() {
        // implementation
    }
});
```

- Extending a class inline:
```java
SomeClass instance = new SomeClass() {
    @Override
    public void someMethod() {
        // override behavior for this one instance
    }
};
```

Common Android examples
- View click listener:
```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // handle click
    }
});
```

- Thread with anonymous Runnable:
```java
new Thread(new Runnable() {
    @Override
    public void run() {
        // background work
    }
}).start();
```

- Anonymous subclass of Thread:
```java
new Thread() {
    @Override
    public void run() {
        // background work
    }
}.start();
```

Scoping rules and "effectively final"
- Anonymous classes can access:
  - Instance fields and methods of the enclosing class (e.g., members of your Activity).
  - Local variables from the enclosing method only if they are *effectively final* (i.e., you don't modify them after assignment).
- Why this restriction? The anonymous class may outlive the method scope; Java captures the value to make behavior predictable.

`this` inside an anonymous class
- Inside the anonymous class, `this` refers to the anonymous instance itself, not the outer class.
- To refer to the enclosing Activity, use `MainActivity.this` (replace `MainActivity` with your Activity class name).

Memory / lifecycle considerations
- Anonymous classes hold an implicit reference to their enclosing instance. For short-lived tasks this is fine.
- For long-running background work attached to an Activity, be mindful of lifecycle and memory leaks. We'll cover safe patterns later.

Quick examples and notes
- Lambda shorthand (Java 8+): for functional interfaces (like `Runnable`), you could write `new Thread(() -> { /* code */ }).start();`. We use full anonymous classes here because they are explicit and clearer for beginners.

Exercises for beginners
1. Replace a named listener with an anonymous one.
2. Create a Thread with an anonymous Runnable that logs 1..5 with 500ms sleep.
3. Try capturing a non-final local variable in an anonymous class and observe the compiler error; then make it effectively final.

---

Timer example using an anonymous class (same behavior as the separate subclass example)
======================================================================================

Goal:
- Demonstrate the timer using an anonymous class variant.
- The timer logs one message per second (use `Log.d`) for 10 seconds.
- Show that the UI thread remains responsive (you can interact with the UI while this runs).

Notes:
- This example uses `SystemClock.sleep(1000)` (convenient for teaching). In production you might prefer `Thread.sleep` with proper interrupt handling or a `ScheduledExecutorService`.

Add this snippet into your `MainActivity.java` (or call it from a button click). Change package names/IDs to match your project.

```java
// MainActivity.java (snippet) - anonymous thread timer example
package your.package.name; // replace with your package

import android.os.Bundle;
import android.os.SystemClock;
import android.util.Log;
import android.view.View;
import android.widget.Button;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "AnonTimerDemo";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main); // ensure your layout exists with the button below

        Button startButton = findViewById(R.id.btnStartTimer); // create a button in your layout

        // Start timer when user taps the button
        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // Anonymous subclass of Thread - the same functionality as a named class,
                // but declared inline where we need it.
                new Thread() {
                    @Override
                    public void run() {
                        Log.d(TAG, "Anonymous timer thread started.");

                        for (int i = 1; i <= 10; i++) {
                            // Log the counter. Log.d is safe to call from background threads.
                            Log.d(TAG, "Timer tick: " + i + "s");

                            // Sleep for 1 second (teaches non-blocking behavior)
                            SystemClock.sleep(1000);
                        }

                        Log.d(TAG, "Anonymous timer thread finished.");
                    }
                }.start(); // start the anonymous thread when button is clicked
            }
        });
    }
}
```

What students should observe:
- When you press the button, the Logcat receives "Timer tick: 1s", "Timer tick: 2s", ...
- The UI stays responsive. You can scroll, press other buttons, rotate the device (if you handle rotation) while the timer runs.
- This proves background work does not block the UI.

Key differences vs named subclass
- Behavior is identical, but the anonymous approach keeps code compact and colocated with the UI action that starts it.
- An anonymous class is ideal for short single-use tasks. For more complex or reusable logic, a named class is preferable.

---

Part 4 — Updating the UI from a background thread (limitations & runOnUiThread)
================================================================================

Why can't background threads touch Views directly?
- All Android UI toolkit classes (Views, Widgets) are **not thread-safe** and must be accessed only from the main (UI) thread.
- If a background thread updates a View directly you'll get an exception:
  - CalledFromWrongThreadException: "Only the original thread that created a view hierarchy can touch its views."
- This rule exists to avoid subtle race conditions and inconsistent view states.

How can a background thread cause a UI update?
- You must ask the UI thread to run code that performs the update. Common ways:
  1. Activity.runOnUiThread(Runnable)
  2. View.post(Runnable) or view.postDelayed(Runnable, ms)
  3. Handler associated with the main Looper (new Handler(Looper.getMainLooper()))
  4. Architecture components (LiveData, ViewModel) or other higher-level patterns (recommended for complex apps)

We will cover `runOnUiThread` now (easy to understand for beginners).

runOnUiThread — concept
- `runOnUiThread(Runnable r)` is a convenience on Activity that posts the Runnable to the main thread's message queue.
- The Runnable will be executed on the main/UI thread, so it's safe to update Views there.

Example: modify the anonymous timer above so it updates a TextView
- We intentionally keep this example short and educational. In Part 5 we will show more robust listener/weak reference patterns and lifecycle-safe approaches.

```java
// MainActivity.java (snippet demonstrating runOnUiThread with anonymous background thread)
package your.package.name;

import android.os.Bundle;
import android.os.SystemClock;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "RunOnUiDemo";
    private TextView timerTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main); // ensure layout has btnStartTimer and timerTextView

        Button startButton = findViewById(R.id.btnStartTimer);
        timerTextView = findViewById(R.id.timerTextView);

        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // Anonymous Runnable inside a new Thread
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Log.d(TAG, "Background thread started.");
                        for (int i = 1; i <= 10; i++) {

                            final int display = i; // must be final/effectively-final to use inside anonymous class

                            // Request UI thread to update the TextView
                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    // This code runs on the main thread — safe to update Views
                                    timerTextView.setText("Timer: " + display + "s");
                                }
                            });

                            SystemClock.sleep(1000);
                        }
                        Log.d(TAG, "Background thread finished.");
                    }
                }).start();
            }
        });
    }
}
```

Explanation of important details:
- `runOnUiThread` queues the Runnable to the main thread. The background thread continues sleeping while the UI update runs on the main thread.
- We use `final int display` (effectively final) because the anonymous inner Runnable captures it.
- `timerTextView.setText(...)` is executed on the UI thread, so it is safe.

Alternatives (brief overview)
- `timerTextView.post(Runnable)` is similar; it posts the Runnable to the view's handler (main thread).
- `Handler` with `Looper.getMainLooper()` allows more control (e.g., scheduling delayed or repeated tasks).
- For production apps, consider lifecycle-aware approaches (ViewModel + LiveData) to avoid leaks and handle configuration changes gracefully.

Common student pitfalls
- Trying to update Views directly from `run()` of a background thread — causes crash.
- Capturing non-final local variables inside anonymous Runnables — compiler error; use effectively-final variables or class fields.
- Forgetting to stop background threads when the Activity is destroyed — can leak memory or crash when thread tries to post to a dead Activity. We'll cover lifecycle-safe cleanup later.

---

Summary and next steps
======================

What we did:
- Explained anonymous classes thoroughly with syntax, scoping, and examples.
- Re-implemented the timer example as an anonymous Thread (or anonymous Runnable) that logs to Logcat (no UI updates), showing the UI remains responsive.
- Explained the UI-thread-only rule and demonstrated `runOnUiThread` as the simplest safe way to update Views from background threads, with code examples.

What to learn next (suggested order):
1. Thread safety basics: `volatile`, interrupts, and safe stop patterns for threads (we'll teach this after the students are comfortable with the above).
2. Lifecycle-aware background work (stopping threads in `onDestroy`, `WeakReference` patterns).
3. `Handler`, `HandlerThread`, and `ScheduledExecutorService` for more robust timers.
4. Modern approaches: ViewModel + LiveData (or Kotlin coroutines) for safe UI updates.

If you'd like, I can:
- Produce a ready-to-drop-in pair of Java files + activity XML for the anonymous-thread timer and the runOnUiThread example.
- Add a follow-up lesson explaining `volatile`, interrupts, and proper thread cancellation patterns (GameTimer stop, `interrupt()` and `join()`).
- Show how to refactor the timer to be lifecycle-aware (stops automatically in `onDestroy`) and avoid leaks.

Which would you like next?