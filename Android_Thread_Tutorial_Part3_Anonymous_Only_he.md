# מדריך אנדרואיד על Threads — חלק 3 (מחלקות אנונימיות) + חלק 4 (עדכון UI עם runOnUiThread)

קובץ זה מלמד על מחלקות אנונימיות ב‑Java (מתאימות לתלמידים מתחילים באנדרואיד), מפרט כיצד לממש טיימר כ‑Thread אנונימי ולוג לג'קט כל שנייה, ואז מסביר כיצד לעדכן את הממשק הגרפי (UI) מתוך Thread באמצעות runOnUiThread.

מבוא:
- חלק 3 — מחלקות אנונימיות: תיאוריה, תחביר, חוקים של סגירה (scoping), דוגמאות נפוצות, ולהסברים מפורטים.
- דוגמת טיימר — יישום הטיימר כ־Thread אנונימי / Runnable אנונימי שמדפיס ל‑Logcat כל שנייה (10 שניות).
- חלק 4 — למה Threads ברקע לא יכולים לשנות את ה‑UI ישירות, וכיצד runOnUiThread פותר את זה.

הערה חשובה לתלמידים:
- בקובץ זה מתמקדים במחלקות אנונימיות וב‑runOnUiThread. נושאים מתקדמים של בטיחות־חוט (volatile, interrupt, סגירת Thread נכון) יגיעו בשיעור נפרד אחרי שהנושאים הבסיסיים יובן היטב.

---

חלק 3 — מחלקות אנונימיות (בפירוט)
==================================

מהי מחלקה אנונימית?
- מחלקה אנונימית היא הגדרה של מחלקה שנוצרת ומאותחלת במקום שבו היא נדרשת, ללא שם מקוטלג.
- לרוב משתמשים בה כאשר רוצים לממש ממשק (interface) או לרשת מחלקה קיימת רק עבור שימוש חוזר קצר במקום אחד.
- באנדרואיד רואים שימוש נרחב במחלקות אנונימיות עבור OnClickListener, Runnable, Comparator ועוד.

מדוע משתמשים בהן?
- מקטינות קובצי קוד ומפחיתות boilerplate כשמימוש קצר וספציפי נדרש במקום אחד.
- משפרות קריאות כשמיישמים את הלוגיקה קרוב לנקודת השימוש.
- מתאימות למשימות קצרות — לא מומלץ להשתמש בהן למשימות ארוכות מאוד שמצריכות ניהול מחזור חיים מורכב.

תחביר בסיסי
- מימוש ממשק אנונימי:
```java
new SomeInterface() {
    @Override
    public void someMethod() {
        // מימוש
    }
};
```

- הרחבת מחלקה אנונימית:
```java
SomeClass instance = new SomeClass() {
    @Override
    public void someMethod() {
        // התנהגות מותאמת עבור מופע זה בלבד
    }
};
```

דוגמאות נפוצות באנדרואיד
- מאזין ללחיצה על כפתור:
```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        // מטפל בלחיצה
    }
});
```

- Runnable אנונימי בתוך Thread:
```java
new Thread(new Runnable() {
    @Override
    public void run() {
        // עבודה ברקע
    }
}).start();
```

- מחלקת Thread אנונימית שמרחיבה Thread:
```java
new Thread() {
    @Override
    public void run() {
        // עבודה ברקע
    }
}.start();
```

חוקי סקופ (Scoping) ו־"effectively final"
- מחלקות אנונימיות יכולות לגשת:
  - לשדות ולשיטות של המחלקה החיצונית (למשל ל־Activity).
  - למשתנים מקומיים של המתודה רק אם הם ״effectively final״ — כלומר לא משתנים אחרי שהוקצו.
- הסיבה: הערך של המשתנה המקומי נלכד על‑ידי המחלקה האנונימית; כדי למנוע חוסר עקביות Java דורשת שהתזמון יהיה ידוע ויציב.

ההבחנה ב‑this בתוך מחלקה אנונימית
- בתוך המחלקה האנונימית `this` מתייחס לאובייקט האנונימי עצמו — לא למחלקה החיצונית.
- כדי לגשת ל־Activity החיצוני יש להשתמש ב‑OuterClassName.this (למשל MainActivity.this).

מניעת דליפות זיכרון (Memory / lifecycle)
- מחלקות אנונימיות אוחזות רפרנס נסתר (implicit) למחלקה החיצונית. אם המחלקה האנונימית ארוכת־חיים (למשל Thread שרץ הרבה זמן), היא עלולה למנוע איסוף אשפה של ה‑Activity ולגרום לדליפה.
- הנחיות:
  - למחלקות קצרות־חיים זה בדרך כלל בסדר.
  - לעבודה ארוכה השתמשו בדפוסי WeakReference, ב־ViewModel/Lifecycle או בסוגים סטטיים שמקבלים WeakReference ל‑Activity.

שאלות נפוצות לתלמידים
- האם ניתן להגדיר constructor במחלקה אנונימית? לא בשם רגיל; אפשר להשתמש ב־instance initializer אם צריך להריץ קוד באתחול.
- מה ההבדל בין אנונימיות ו‑lambda? ל־lambda יש תחביר קצר יותר ומותאם ל־functional interfaces (Runnable, OnClickListener), אך עבור תלמידים חשוב לראות את המבנה המלא של המחלקה האנונימית לפני המעבר ל‑lambda.

תרגילים מומלצים
1. החליפו מאזין בשם במאזין אנונימי לכפתור.
2. צרו Thread עם Runnable אנונימי שמדפיס 1..5 בהשהיית 500ms. בזמן הריצה לחצו על כפתור ובדקו שה‑UI לא נתקע.
3. נסו ללכוד משתנה מקומי שאינו final וראו את שגיאת הקומפילציה — לאחר מכן השאירו אותו כאפקטיבית־פינלית ותראו שהשגיאה נעלמת.

---

דוגמת טיימר באמצעות מחלקה אנונימית (לוג בלבד, לא עדכון UI)
============================================================

מטרה:
- להמחיש יצירת Thread אנונימי שמדפיס ל־Logcat כל שנייה במשך 10 שניות.
- להדגים שה‑UI נשאר רספונסיבי.

שים לב:
- בדוגמה זו נשתמש ב‑SystemClock.sleep או Thread.sleep — שניהם טובים ללימוד. בייצור מומלץ לנהל interruption טוב יותר.

הוסיפו את הקוד הבא בתוך MainActivity (שורה בתוך onCreate או בתוך מאזין לכפתור):

```java
// MainActivity.java (קטע)
package your.package.name; // החליפו בשם החבילה שלכם

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
        setContentView(R.layout.activity_main); // ודאו שיש לכם layout מתאים

        Button startButton = findViewById(R.id.btnStartTimer);

        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // Thread אנונימי — מוחלף למופע אחד בלבד, במקום בו הוא צריך להמצא
                new Thread() {
                    @Override
                    public void run() {
                        Log.d(TAG, "Anonymous timer thread started.");

                        for (int i = 1; i <= 10; i++) {
                            // לוג ל‑Logcat — קל לבדוק ולראות שאין חסימה ב‑UI
                            Log.d(TAG, "Timer tick: " + i + "s");

                            // השהייה של שנייה — לא חוסמת את ה‑UI, רק את ה‑Thread הזה
                            SystemClock.sleep(1000);
                        }

                        Log.d(TAG, "Anonymous timer thread finished.");
                    }
                }.start(); // התחלת ה‑Thread האנונימי
            }
        });
    }
}
```

מה תלמידים יבחינו:
- בעת לחיצה על הכפתור, Logcat יציג טיקים 1s..10s.
- ה‑UI נשאר רספונסיבי — ניתן ללחוץ על כפתורים אחרים, לגלול במסך וכו'.
- המחלקה האנונימית שומרת על קוד קצר וקולח במקום בו אנחנו זקוקים לה.

הבדלים מול מחלקה בשם נפרדת
- אין שינוי בהתנהגות; היתרון כאן הוא קוד רב פחות וקירבה לנקודת השימוש.
- למחלקות מורכבות או שנרצה לממש לוגיקה חוזרת — עדיף שם מפורש בקובץ נפרד.

---

חלק 4 — עדכון UI מתוך Thread ברקע (מגבלות ו‑runOnUiThread)
=============================================================

למה לא ניתן לגשת ל‑Views מתוך Thread ברקע?
- ספריית ה‑UI של אנדרואיד אינה thread‑safe. כל הקריאות ל־Views חייבות להתרחש על ה‑main/UI thread.
- אם Thread ברקע ינסה לקרוא או לכתוב על View תיתקל בשגיאה: CalledFromWrongThreadException ("Only the original thread that created a view hierarchy can touch its views").
- מטרת הכלל: למנוע race conditions ושגיאות עקב גישה סימולטנית לנתוני UI.

כיצד נגרום לעדכון UI מ‑Thread ברקע?
- צריך לבקש מה‑UI thread לבצע את הקוד שמעדכן את ה‑View. דרכים נפוצות:
  1. Activity.runOnUiThread(Runnable)
  2. view.post(Runnable) או view.postDelayed(Runnable, ms)
  3. Handler gekoppeld ל־Looper.getMainLooper()
  4. שימוש ברכיבים מודרניים: LiveData + ViewModel, או שימוש ב־WorkManager וכו' (מומלץ בפרויקטים גדולים)

הסבר על runOnUiThread
- שיטה נוחה על Activity שמכניסה Runnable לתור ההודעות של ה‑main thread.
- ה‑Runnable ירוץ על ה‑UI thread, כך עדכון Views בו יהיה בטוח.

דוגמה — עדכון TextView מתוך ה‑Thread האנונימי שנריץ:

```java
// MainActivity.java (קטע להמחשה)
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
        setContentView(R.layout.activity_main);
        Button startButton = findViewById(R.id.btnStartTimer);
        timerTextView = findViewById(R.id.timerTextView);

        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Log.d(TAG, "Background thread started.");
                        for (int i = 1; i <= 10; i++) {
                            final int display = i; // צריך להיות final או effectively-final

                            // מבקשים מה‑UI thread לעדכן את ה־TextView
                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    // ריצה על ה‑main thread — בטוח לעדכן Views כאן
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

נקודות חשובות:
- ה‑Runnable שמועבר ל‑runOnUiThread ירוץ על ה‑UI thread.
- ההשהייה (sleep) מתבצעת ב‑Thread ברקע, ולכן לא חוסמת את ה‑UI.
- יש להשתמש ב‑final/effectively-final כאשר לוכדים ערכים מקומיים בתוך ה‑Runnables.

חלופות קצרות:
- `timerTextView.post(Runnable)` — מתקבל אותו אפקט: הפונקציה תפורסם ל־handler של ה‑View.
- `new Handler(Looper.getMainLooper()).post(Runnable)` — גישה גמישה יותר לשליטה ולהשהיות.

טעויות נפוצות אצל תלמידים
- ניסיון לקרוא/לכתוב ל־View ישירות מתוך run() של Thread — מוביל ל־Crash.
- לכידת משתנים שאינם אפקטיבית־פינליים בתוך אנונימיות — שגיאת קומפילציה.
- השארת Threads ארוכים בלי סיום — עלולות לגרום לדליפות זיכרון. נעבור לנושאים אלה בפרק Best Practices.

---

סיכום והצעות להמשך
=====================

מה עשינו:
- הסבר מעמיק על מחלקות אנונימיות ב‑Java עם דוגמאות רבות.
- מימוש טיימר כאנונימית (Thread/Runnable) שמדפיס ל־Logcat והדגמת חוסר חסימה של ה‑UI.
- הסבר על הכלל שאוסר על גישה ישירה ל‑Views מתוך Threads ברקע והדגמת שימוש ב‑runOnUiThread לעדכון UI בבטחה.

מה ללמוד הלאה (הצעה בסדר לימוד מומלץ):
1. בטיחות חוטים: volatile, interrupts, עצירת thread בצורה נקיה.
2. ניהול מחזור‑חיים: עצירת Threads ב‑onDestroy, שימוש ב‑WeakReference למניעת דליפות.
3. Handler/HandlerThread ו‑ScheduledExecutorService לטיימרים מדויקים.
4. גישות מודרניות: ViewModel + LiveData (או Kotlin coroutines) לעבודה בטוחה מול UI.

אם תרצו, אשמח:
- להכין קבצי Java ו‑XML מוכנים להעתקה לפרויקט (דוגמת טיימר אנונימי + runOnUiThread).
- להמשיך ולספק שיעור מעשי על volatile ו‑interrupts, ולאחר מכן לעדכן את דוגמאות ה‑GameTimer כדי להיות בטוחות מבחינת מחזור חיים.
- לתרגם גם את הקבצים המוכנים ולעדכן אותם בריפוזיטורי כשאתם מאשרים.

שיהיה בהצלחה ולימוד מהנה!