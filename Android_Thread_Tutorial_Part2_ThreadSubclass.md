# Android Codelab — Threads from Scratch (Java, Beginner Level)

This codelab teaches Android students how to use Java Threads step‑by‑step, starting from a named class that extends `Thread`.  
You will learn how to:

1. Create a simple `Thread` subclass that logs a counter every second for 10 seconds.
2. Start that thread from your `Activity` and observe that the UI is not blocked.
3. Turn the thread into a game timer that runs until you stop it (start-on-first-move, stop-on-game-over) using a loop.
4. Properly stop the thread and clean up in the Activity lifecycle.
5. Prepare to advance to anonymous threads (next lesson).

This lesson focuses on clarity and pedagogy. Advanced thread-safety topics (e.g., `volatile`, `interrupt` nuances) are saved for the "Best Practices" follow-up after students are comfortable with the basics.

---

## Part A — Theory: Why Threads?

- Android apps have a single main thread, called the UI thread. Any long-running task on the UI thread (sleep, heavy computation, network I/O) will freeze the UI and make the app appear unresponsive.
- Threads let us move work off the UI thread so the UI remains responsive.
- A `Thread` runs code in parallel (concurrently) with the UI thread. For simple teaching, we'll create named classes that extend `Thread`.

Key rule to remember:
- Background threads must not update UI elements directly. We'll use `Log.d` in this lesson to show behavior without touching UI. (Updating UI from background threads comes in later lessons.)

---

## Part B — Create a Thread subclass: timer for 10 seconds

We create a simple class `SimpleTimerThread` that extends `Thread`. It logs a counter (1..10) to Logcat once per second.

File: `SimpleTimerThread.java`
```java
package your.package.name; // <-- change to your app package

import android.os.SystemClock;
import android.util.Log;

/**
 * SimpleTimerThread
 *
 * A very small Thread subclass that demonstrates background work.
 * It logs "Timer tick: Ns" once per second for 10 seconds.
 *
 * Teaching goals:
 * - Show how to create a Thread subclass.
 * - Demonstrate that sleeping in this thread does NOT block the UI thread.
 */
public class SimpleTimerThread extends Thread {
    private static final String TAG = "SimpleTimerThread";

    // default constructor (inherited)
    public SimpleTimerThread() {}

    @Override
    public void run() {
        Log.d(TAG, "SimpleTimerThread started.");
        // Count from 1 to 10, logging each second.
        for (int i = 1; i <= 10; i++) {
            // Log from background thread is safe
            Log.d(TAG, "Timer tick: " + i + "s");

            // Sleep for 1 second to simulate a timer tick.
            // Using SystemClock.sleep makes it simple for teaching.
            SystemClock.sleep(1000);
        }
        Log.d(TAG, "SimpleTimerThread finished.");
    }
}
```

How to use it from `MainActivity`:

```java
// MainActivity.java (snippet)
SimpleTimerThread timer = new SimpleTimerThread();
timer.start(); // spawns a new background thread that runs run()
```

What students should do:
- Add `SimpleTimerThread` to the project.
- Start it from `onCreate()` or from a button click.
- Open Logcat and filter by tag `SimpleTimerThread` to see the ticks.
- While it's running, interact with the UI (press buttons, scroll) — UI should remain responsive.

---

## Part C — From fixed-duration to an open-ended game timer

Real games don't last a fixed 10 seconds. A typical game timer should:
- Start when the game starts (for example, after the player's first move).
- Continue running until the game ends (win/tie/quit).
- Stop cleanly when the game is over or when the Activity is destroyed.

To implement this, we convert the simple for-loop timer to a loop that runs while a `running` flag is true.

File: `GameTimerThread.java`
```java
package your.package.name; // <-- change to your app package

import android.os.SystemClock;
import android.util.Log;

/**
 * GameTimerThread
 *
 * A background timer thread intended to run for an arbitrary duration,
 * from the moment the game starts until the game finishes.
 *
 * Usage:
 * - Create instance but do not start until first move.
 * - Call start() to begin.
 * - Call stopTimer() (from the UI thread) when the game is over.
 *
 * Note: This implementation focuses on teaching the pattern.
 * Best-practice thread-safety (volatile/interrupt) will be discussed later.
 */
public class GameTimerThread extends Thread {
    private static final String TAG = "GameTimerThread";

    // Flag that indicates whether the timer should keep running.
    // We'll demonstrate a simple boolean field first. Later lessons cover volatile.
    private boolean running = true;

    // Seconds elapsed since timer started
    private int secondsElapsed = 0;

    public GameTimerThread() {
        // default constructor
    }

    @Override
    public void run() {
        Log.d(TAG, "GameTimerThread started.");

        // Keep looping until someone sets running = false
        while (running) {
            // Sleep for 1 second to simulate ticking
            SystemClock.sleep(1000);
            secondsElapsed++;

            // Log the elapsed time (safe from background thread)
            Log.d(TAG, "Game time: " + secondsElapsed + "s");

            // Optionally: break early if desired (but we rely on running flag)
        }

        Log.d(TAG, "GameTimerThread exiting after " + secondsElapsed + "s");
    }

    /**
     * Request the timer to stop. Call this from the main (UI) thread when the game ends.
     * This method sets running=false so the while loop exits naturally.
     * In a later lesson we'll mark this field volatile and also interrupt the thread to wake it from sleep.
     */
    public void stopTimer() {
        running = false;
        // We could call interrupt() here to wake the thread if it's sleeping.
        // We'll teach interrupts later.
    }

    public int getSecondsElapsed() {
        return secondsElapsed;
    }
}
```

Important design choices explained:
- We use `while (running)` because the game's duration is unknown. `for` with fixed iterations won't work.
- `stopTimer()` sets the flag so the thread exits the loop on its next iteration.
- We log the elapsed time each second — useful for debugging/testing.
- We intentionally avoid UI updates from this thread in this lesson.

---

## Part D — Where and when to start and stop the timer in the Activity

Common pattern:
- Start the timer only once, when the game actually begins (after the first move).
- Use a boolean guard in your Activity (e.g., `timerStarted`) to ensure you start it only once.
- Stop the timer when the game ends (win/tie/quit) by calling `gameTimer.stopTimer()`.

Example `MainActivity` wiring (snippets):

```java
// MainActivity.java (snippets)
public class MainActivity extends AppCompatActivity {
    private GameTimerThread gameTimer;
    private boolean timerStarted = false;

    // Called when user clicks on a cell (example)
    public void onCellClick(View view) {
        // handle user move...
        // After placing the first move, start the timer:
        if (!timerStarted) {
            gameTimer = new GameTimerThread();
            gameTimer.start();
            timerStarted = true;
        }
    }

    // When the model/detector signals game over:
    private void onGameOver() {
        if (gameTimer != null) {
            gameTimer.stopTimer(); // request the thread to exit cleanly
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Ensure we stop the timer when the Activity is destroyed to avoid background threads lingering
        if (gameTimer != null) {
            gameTimer.stopTimer();
        }
    }
}
```

Notes:
- `stopTimer()` only requests a stop; the thread will exit its loop and return from `run()`. If you need immediate wake-up (instead of waiting for the next sleep), we will cover `interrupt()` later.
- If you want to wait for the thread to finish before continuing, you can call `gameTimer.join()` from a background or lifecycle-safe place — but avoid calling `join()` on the UI thread because it blocks.

---

## Part E — Demonstration & Testing

1. Add `SimpleTimerThread` and `GameTimerThread` classes to your project.
2. In `MainActivity`, wire a button or your `onCellClick` to start the `SimpleTimerThread` (for the simple 10-second demo).
3. Observe Logcat:
   - With `SimpleTimerThread`, you'll see "Timer tick: 1s" ... "Timer tick: 10s".
   - With `GameTimerThread`, start it after a first move and then stop it when the game ends — observe the ticks.
4. While the thread is running, interact with the UI (press buttons, type text, scroll). The UI should remain responsive.

Exercise idea for students:
- Start `GameTimerThread` on the first click of the board and stop it when the model's `checkWin()` returns a winner. Log the time in Logcat and also print a Toast with the time (displaying a Toast is allowed from UI thread; you would call it from the code that handled the game over on the UI thread).

---

## Part F — Common pitfalls and teacher tips

- Do NOT update Views (TextView.setText, Button.setEnabled, etc.) from the thread's `run()` method — that will crash the app. We'll show safe UI updates in the next lesson (anonymous threads + runOnUiThread).
- Avoid calling `Thread.sleep()` on the UI thread — this will freeze the app.
- Keep threads short-lived or stop them in `onDestroy()` to avoid memory leaks.
- For long-lived background work consider using `HandlerThread`, `Executors`, or modern architecture components in future lessons.

---

## Part G — Transition to anonymous classes

After students understand this named-subclass approach, the next lesson will show how to implement the *same* behavior with an **anonymous Thread / anonymous Runnable** declared inline in the Activity (no separate class file). This is useful for compact code where the logic is used in one place (like a button click). The anonymous approach will:

- Re-create the 10-second timer as an anonymous class that logs to Logcat.
- Re-create the open-ended game timer as an anonymous class with a `running` flag, started on first move and stopped on game over.

Students will then compare the two patterns (named class vs anonymous) and understand when to use each.

---

## Part H — Best Practices Teaser (full details later)

We intentionally used a plain boolean flag for simplicity. For production and correct thread-safety you should:

- Mark flags shared between threads as `volatile` (so writes from one thread are visible to others).
- Consider interrupting sleeping threads to wake them immediately when stop is requested.
- Avoid referencing Activity UI directly from background threads; pass notifications to the UI thread using `runOnUiThread`, `Handler`, or architecture components.
- Stop background threads in `onDestroy()` or use lifecycle-aware components (ViewModel + LiveData or Services) for long-running tasks.

We will cover these details in an upcoming "Best Practices & Thread Safety" lesson.

---

## Appendix — Full example files (copy/paste)

1) `SimpleTimerThread.java` (same as above)  
2) `GameTimerThread.java` (same as above)  
3) `MainActivity.java` (snippet):

```java
// MainActivity.java (compact example with start-on-first-move and stop-on-game-over)
package your.package.name;

import android.os.Bundle;
import android.view.View;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    private GameTimerThread gameTimer;
    private boolean timerStarted = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    // Example: called when user clicks any board cell
    public void onCellClick(View view) {
        // After processing the move in your model...
        if (!timerStarted) {
            gameTimer = new GameTimerThread();
            gameTimer.start();
            timerStarted = true;
        }

        // ... other game logic ...
    }

    // Call this when game ends (from UI thread)
    private void onGameOver() {
        if (gameTimer != null) {
            gameTimer.stopTimer();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (gameTimer != null) {
            gameTimer.stopTimer();
        }
    }
}
```

---

## Exercises (for students)

1. Implement `SimpleTimerThread`, start it on a button click, and verify Logcat output. While it runs, click other UI elements to show the UI is not blocked.
2. Implement `GameTimerThread`. Start it only after the first move and stop it on game over. Log the elapsed seconds.
3. Modify `GameTimerThread.stopTimer()` to call `this.interrupt()` after setting `running=false`. Observe behavior (we'll explain interrupts later).
4. Add a method `getSecondsElapsed()` and display the final elapsed time in a Toast when the game ends (remember Toast must be shown from the UI thread).

---

## Closing Notes

This codelab gives students a step-by-step path:
- Start with a clear, named Thread subclass (easy to reason about).
- Move to an open-ended timer using a loop and a stop flag.
- Prepare to re-implement the same behavior using anonymous classes (next lesson) for compactness.

When you're ready, I'll prepare:
- The anonymous-class version (Part 3) with equivalent code and explanations.
- A follow-up lesson about `volatile`, `interrupt`, and safe thread termination with code examples and exercises.

Happy teaching and coding!