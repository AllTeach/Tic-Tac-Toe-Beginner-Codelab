# קורס מעשי: Threads מאפס (Android, Java — רמת מתחילים)

מדריך זה מלמד את תלמידיכם צעד‑אחר‑צעד איך להשתמש ב‑Threads ב‑Android באמצעות Java, מתוך דגש חינוכי ובהדרגה:
1. יצירת מחלקה שמרחיבה Thread שמדפיסה ספירה ללוג (1..10), עם השהייה של שנייה.
2. התחלת ה‑Thread מתוך ה‑Activity והדגמה שה‑UI לא נחסם.
3. שינוי ה‑Thread לטיימר למשחק שממשיך לרוץ עד שנעצר (start-on-first-move, stop-on-game-over) באמצעות לולאה.
4. עצירת ה‑Thread וניקוי במשימות ה‑Activity (lifecycle).
5. הכנה למעבר להמחשת אותו טיימר כ‑anonymous class בשיעור הבא.

נשמור על הבהרה ופדגוגיה — נושאים מתקדמים של בטיחות חוטים (כגון volatile, interrupt) יגיעו בשיעור נפרד אחרי שהבסיס ברור.

---

חלק א' — תאוריה קצרה: למה Threads?
- ל‑Android יש שרשור ראשי אחד — ה‑UI thread. כל משימה ארוכה על ה‑UI thread (שינה, חישוב כבד, I/O) תגרום לקפיאה של הממשק.
- Threads מאפשרים להריץ עבודה ברקע ולהשאיר את ה‑UI רספונסיבי.
- Thread מריץ קוד במקביל ל‑UI thread. לשם ההוראה ניצור מחלקות השמות שמרחיבות Thread.

חוק חשוב לזכור:
- Threads ברקע אינם מעדכנים UI ישירות. בשיעורים אלה נשתמש ב‑Log.d כדי להראות פעילות ללא גישה ל‑Views. עדכון UI ילמד בשיעורים הבאים.

---

חלק ב' — צור מחלקת Thread: טיימר ל‑10 שניות

ניצור מחלקה בשם `SimpleTimerThread` שמרחיבה `Thread`. היא תדפיס ללוג ("Timer tick: Ns") פעם בשנייה במשך 10 שניות.

קובץ: `SimpleTimerThread.java`

```java
package your.package.name; // <-- שנה לשם החבילה שלך

import android.os.SystemClock;
import android.util.Log;

/**
 * SimpleTimerThread
 *
 * Thread פשוט שמדגים עבודה ברקע:
 * מדפיס "Timer tick: Ns" כל שנייה במשך 10 שניות.
 *
 * מטרת הלימוד:
 * - להראות איך לרשת Thread ולהריץ run().
 * - להראות שה‑sleep בתוך ה‑Thread לא חוסם את ה‑UI thread.
 */
public class SimpleTimerThread extends Thread {
    private static final String TAG = "SimpleTimerThread";

    public SimpleTimerThread() {
        // קונסטרקטור ברירת מחדל
    }

    @Override
    public void run() {
        Log.d(TAG, "SimpleTimerThread started.");
        // ספירה מ‑1 עד 10, עם השהייה של שנייה בין כל איטרציה
        for (int i = 1; i <= 10; i++) {
            // קריאה ל‑Log.d בטוחה מתוך Thread ברקע
            Log.d(TAG, "Timer tick: " + i + "s");

            // השהייה של שנייה — SystemClock.sleep פשוטה ללימוד
            SystemClock.sleep(1000);
        }
        Log.d(TAG, "SimpleTimerThread finished.");
    }
}
```

כיצד להשתמש בזה מתוך `MainActivity`:

```java
// MainActivity.java (קטע)
SimpleTimerThread timer = new SimpleTimerThread();
timer.start(); // יוצר Thread חדש שמריץ את run() ברקע
```

מה התלמידים צריכים לעשות:
- להוסיף את `SimpleTimerThread` לפרויקט.
- להפעילו מתוך `onCreate()` או כפתור.
- לפתוח את Logcat ולסנן לפי TAG `SimpleTimerThread` כדי לראות את הטיקים.
- בזמן שה‑Thread רץ, לנסות להשתמש ב‑UI — הוא צריך להישאר רספונסיבי.

---

חלק ג' — מ‑טיימר בעומד ל‑טיימר פתוח למשחק

משחק לא נגמר אחרי 10 שניות בהכרח — לכן נהפוך את הלולאה ללולאה שתמשיך עד שמבקשים ממנה להיפסק.

קובץ: `GameTimerThread.java`

```java
package your.package.name; // <-- שנה לשם החבילה שלך

import android.os.SystemClock;
import android.util.Log;

/**
 * GameTimerThread
 *
 * טיימר שמריץ את עצמו לאורך זמן בלתי ידוע — מתחילת המשחק ועד סיומו.
 *
 * שימוש:
 * - צור את המופע אך אל תתחיל עד שהמשחק מתחיל (למשל אחרי המהלך הראשון).
 * - קרא start() כדי להתחיל.
 * - קרא stopTimer() מה‑UI thread כדי לעצור כשהמשחק הושלם.
 *
 * הוראה: לפרק זה נתמקד בדפוס. פירוט של volatile/interrupt יבוא בשיעור מתמשך.
 */
public class GameTimerThread extends Thread {
    private static final String TAG = "GameTimerThread";

    // דגל שמציין אם ה‑Thread ממשיך לרוץ
    private boolean running = true;

    // שניות שחלפו מאז התחלה
    private int secondsElapsed = 0;

    public GameTimerThread() {
        // קונסטרקטור ברירת מחדל
    }

    @Override
    public void run() {
        Log.d(TAG, "GameTimerThread started.");

        // לולאה רצה כל עוד ריצת הטיימר רצויה
        while (running) {
            // השהייה של שנייה
            SystemClock.sleep(1000);
            secondsElapsed++;

            // לוג התקדמות (בטוח מתוך Thread ברקע)
            Log.d(TAG, "Game time: " + secondsElapsed + "s");

            // אם רצית, ניתן להוסיף תנאים לביצוע משימות נוספות
        }

        Log.d(TAG, "GameTimerThread exiting after " + secondsElapsed + "s");
    }

    /**
     * בקשה לעצירת הטיימר — קוראים מה‑UI thread כשהמשחק נגמר.
     * השיטה משנה את הדגל כך שהלולאה תצא באופן טבעי.
     * בשיעורים הבאים נדון בצפייה ו‑interrupt כדי להקפיץ את ה‑Thread משינה.
     */
    public void stopTimer() {
        running = false;
        // אפשר לקרוא this.interrupt(); כדי להעיר את ה‑Thread משינה — ילמד בהמשך.
    }

    public int getSecondsElapsed() {
        return secondsElapsed;
    }
}
```

הסבר לעיצוב:
- השתמשנו ב‑while (running) כי אורך המשחק אינו ידוע מראש.
- stopTimer() מבקשת עצירה; ה‑Thread יצא מהלולאה בסיבוב הבא.
- היינו מדלגים על עדכון UI מתוך ה‑Thread בשיעור זה.

---

חלק ד' — מתי להתחיל ולעצור את הטיימר בתוך ה‑Activity

דפוס מקובל:
- התחל את הטיימר רק פעם אחת, כשהמשחק מתחיל (למשל לאחר המהלך הראשון).
- השתמש בדגל ב‑Activity (לדוגמה `timerStarted`) כדי להבטיח התחלה פעם אחת.
- עצור את הטיימר כאשר המשחק מסתיים ע"י קריאה ל‑`gameTimer.stopTimer()`.

קטעי קוד ב‑`MainActivity`:

```java
// MainActivity.java (קטעים)
public class MainActivity extends AppCompatActivity {
    private GameTimerThread gameTimer;
    private boolean timerStarted = false;

    // נקרא כאשר משתמש לוחץ על תא בלוח (דוגמה)
    public void onCellClick(View view) {
        // בצע את מה שתצטרכו מבחינת המודל...
        // אחרי המהלך הראשון, נתחיל את הטיימר:
        if (!timerStarted) {
            gameTimer = new GameTimerThread();
            gameTimer.start();
            timerStarted = true;
        }
    }

    // נקרא כשזוהה סוף משחק
    private void onGameOver() {
        if (gameTimer != null) {
            gameTimer.stopTimer(); // מבקשים מה‑Thread לעצור
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // לוודא עצירה של ה־Thread כשמפעילים השמדה של ה־Activity
        if (gameTimer != null) {
            gameTimer.stopTimer();
        }
    }
}
```

הערות:
- `stopTimer()` רק מבקש עצירה — ה‑Thread יוצא בלולאה ומסיים.
- אם רוצים עצירה מידית (ולא להמתין לסיבוב הבא), נעשה שימוש ב‑`interrupt()` — ילמד בהמשך.
- המנעות מ‑join() על ה‑UI thread: אל תשתמשו ב‑join() מה‑UI thread כי זה יחסום את הממשק.

---

חלק ה' — הדגמה ובדיקות

1. הוסיפו `SimpleTimerThread` ו־`GameTimerThread` לפרויקט.
2. ב‑`MainActivity` חברו כפתור שיתחיל את `SimpleTimerThread` (למחשה של 10 שניות).
3. פתחו את Logcat וצפו ב:
   - `SimpleTimerThread`: "Timer tick: 1s" ... "Timer tick: 10s".
   - `GameTimerThread`: התחלה בסיבוב ראשון וסגירה בסוף המשחק.
4. בזמן הריצה של ה‑Thread, נסו להשתמש ב‑UI (לחצן, גלילה) — ה‑UI צריך להישאר רספונסיבי.

תרגיל מומלץ:
- התחילו את `GameTimerThread` אחרי הלחיצה הראשונה על הלוח, וסיימו אותו כש‑`checkWin()` במודל מזהה מנצח. בעת סיום, הדפיסו ללוג או הציגו Toast עם זמן המשחק (Toast צריך להיקרא מה‑UI thread; ניתן לקרוא אותו ממקום שבו אתם מטפלים בסיום המשחק על ה‑UI thread).

---

חלק ו' — תקלות נפוצות וטיפים למורים

- אל תעדכנו Views מתוך `run()` של ה‑Thread — זה יגרום לקריסה. עדכון UI נלמד בשיעורים הבאים.
- אל תקראו ל‑Thread.sleep() על ה‑UI thread — זה יחסום את האפליקציה.
- עצרו Threads ב‑onDestroy() כדי למנוע דליפות זיכרון.
- לעבודה ארוכת טווח שקלו `HandlerThread`, `Executors`, או רכיבים מודרניים (ViewModel + LiveData) בעתיד.

---

חלק ז' — מעבר למחלקות אנונימיות

כשיתבססו על הגישה הזו, בשיעור הבא נראה איך לממש את אותו טיימר כמחלקה אנונימית (anonymous Thread / Runnable) בתוך ה‑Activity — קוד קצר ונוח כשברצוננו לבצע לוגיקה במקום אחד. התלמידים ישוו בין שתי הגישות ויבינו מתי מומלץ כל דפוס.

---

חלק ח' — טעימה של Best Practices (יושלם בהמשך)
- בשיעור זה השתמשנו ב‑boolean רגיל לפשטות. לפרודקשן יש להשתמש ב:
  - `volatile` לדגלים המשותפים בין Threads, כדי להבטיח נראות שינויים.
  - `interrupt()` להעיר Threads ישנים משינה ולהגיב בהגדרה ב‑catch של InterruptedException.
  - הימנעות מרפרנסים חזקים ל‑Activity מתוך Threads ארוכי־חיים; שימוש ב‑WeakReference או רכיבים מבוססי lifecycle.
- נרחיב על זה בשיעור "בטיחות חוטים ונוהלי עבודה נכונים".

---

נספח — דוגמאות קבצים מלאים (להעתקה)

1) `SimpleTimerThread.java` (ראה מעלה)  
2) `GameTimerThread.java` (ראה מעלה)  
3) `MainActivity.java` (דוגמה מקוצרת):

```java
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

    // דוגמה: נקרא כאשר המשתמש לוחץ על תא בלוח
    public void onCellClick(View view) {
        // אחרי עדכון המודל...
        if (!timerStarted) {
            gameTimer = new GameTimerThread();
            gameTimer.start();
            timerStarted = true;
        }
    }

    // קריאה בסיום המשחק (על ה‑UI thread)
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

תרגילים לסטודנטים
1. יישמו `SimpleTimerThread`, התחילו אותו בעת לחיצה על כפתור ובדקו את Logcat. בזמן הריצה לחצו על UI — הוכחת שלא נחסם.
2. יישמו `GameTimerThread`. התחילו אותו לאחר המהלך הראשון וסקו אותו בסיום המשחק. רישמו שניות לעוגן ותציגו ב‑Toast את הזמן בסיום (הצגת Toast תיעשה על ה‑UI thread).
3. נסו להוסיף `this.interrupt()` ל־`stopTimer()` ולבדוק מה קורה — נסביר interrupts בשיעור הבא.
4. הוסיפו `getSecondsElapsed()` והציגו את זמן המשחק האחרון ב‑Toast כשמסיימים.

---

סיכום
- מדריך זה נותן דרך מובנית ללמד Threads בתחילת קורס אנדרואיד: מתחילים במחלקה בשם שמרחיבה Thread, עוברים לטיימר פתוח עם stop-flag, ומכינים לתרגולים עם מחלקות אנונימיות.
- כשתרצו, אכין את שיעור ההמשך: מימוש אנונימי וטכניקות עדכון UI (runOnUiThread, Handler), ואחר כך שיעור על `volatile`/interrupt/cleanup.

בהצלחה בהוראה!