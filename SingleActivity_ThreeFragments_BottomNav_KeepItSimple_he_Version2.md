# מדריך מעשי לאנדרואיד — Single Activity, שלושה Fragments ו‑Bottom Navigation (שמרו על פשטות)

מדריך זה מלמד באופן הפשוט ביותר לתלמידים מתחילים:
- Activity יחידה (MainActivity)
- שלושה Fragments (Fragment1, Fragment2, Fragment3)
- ניווט תחתון (BottomNavigationView) להחלפה בין ה‑Fragments
- בכל Fragment יש כפתור (Button) ו‑TextView; לחיצה על הכפתור מעדכנת את ה‑TextView
- כל החיפוש אחרי Views נעשה בעזרת findViewById (ללא ViewBinding)

מטרה: להכיר את מחזור החיים של Fragment (onCreateView לעומת onViewCreated) וכיצד להחליף Fragments בעזרת FragmentManager. נשמור על הקוד וה‑UX מינימליים לצורכי הוראה.

---

## סקירה מהירה — מה ילמדו התלמידים ולמה

- מהו Fragment וכיצד הוא קשור ל‑Activity
- מחזור חיי ה‑View של ה‑Fragment (למה מאמצים inflate ב‑onCreateView ומפעילים UI ב‑onViewCreated)
- איך FragmentManager מחליף Fragments (replace לעומת add/hide)
- איך BottomNavigationView עובד וכיצד לטפל בניווט בפשטות
- נושאים בסיסיים של state: מה קורה בסיבוב מסך (rotation) ולמה שמרנו על תנהגות פשוטה
- דרכים פשוטות לתקשורת Activity ↔ Fragment
- נגישות וטיפים קטנים לטובות מעשיות עבור מתחילים

---

## 1 — הוספת התלות ב‑Material

בקובץ `build.gradle` של המודול (Module: app) הוסיפו את הספריה של Material אם היא לא קיימת:

```gradle
dependencies {
    implementation (libs.material)
    // ... שאר התלויות
}
```

סנכרנו את הפרויקט.

---

## 2 — מבנה פרויקט (פשוט)

```
app/
├─ java/
│   └─ com.example.bottomnavdemo/
│       ├─ MainActivity.java
│       ├─ Fragment1.java
│       ├─ Fragment2.java
│       └─ Fragment3.java
├─ res/
│   ├─ layout/
│   │   ├─ activity_main.xml
│   │   ├─ fragment_1.xml
│   │   ├─ fragment_2.xml
│   │   └─ fragment_3.xml
│   ├─ menu/
│   │   └─ bottom_nav_menu.xml
│   └─ values/
│       └─ strings.xml
```

---

## 3 — מהו Fragment? (תיאוריה)

Fragment הוא חתיכת ממשק (UI) שניתנת לשימוש חוזר ויש לה מחזור חיים משלה, והיא מאוחסנת בתוך Activity. אפשר לראות ב‑Fragment כבלוק מודולרי שמאפשר להציג מסכים שונים בתוך אותה Activity.

למה משתמשים ב‑Fragments?
- לשימוש חוזר של רכיבי UI על מסכים ובגדלים שונים (טלפון מול טאבלט).
- לבניית ממשקים גמישים — אפשר להחליף חלקים במסך מבלי לטעון Activity חדש.
- כדי לשמור על לוגיקה מסך נפרדת ומודולרית.

חשוב: ל‑Fragment יש גם מחזור חיים של האובייקט וגם מחזור חיים של ה‑View שלו. ה‑View של Fragment יכול להיות מושמד וייווצר מחדש בעוד האובייקט Fragment עצמו עדיין קיים — לכן יש להיזהר ברפרנסים ל‑View.

---

## 4 — מחזור חיי ה‑View ב‑Fragment (מה חשוב לדעת)

המתודות החשובות עבור המדריך הזה:

- onAttach() — ה‑Fragment מצטרף ל‑Activity
- onCreate() — יצירת מופע ה‑Fragment (מתאים לאחסון state שאינו view)
- onCreateView() — כאן מטמיעים (inflate) את פריסת ה‑Fragment ומחזירים את ה‑View
  - אינך אמור לגשת子 ל‑child views כאן (יש מצב שעדיין לא נוצרו)
- onViewCreated() — נקרא לאחר onCreateView; כאן כבר בטוח לקרוא findViewById ולרגיש listeners
- onStart(), onResume() — ה‑Fragment גלוי ומקיים אינטראקציה
- onPause(), onStop() — שינויים במצב הוויזיביליות
- onDestroyView() — view ה‑Fragment מושמד; יש לנקות רפרנסים ל‑View כאן
- onDestroy() — מופע ה‑Fragment מושמד

טיפ חשוב להוראה:
- אל תשמרו רפרנסים ל‑Viewות בערכים שיחזיקו בזמן ארוך יותר מ‑onDestroyView — נקה אותם ב‑onDestroyView כדי למנוע דליפות זיכרון.

---

## 5 — יסודות ה‑Bottom Navigation (תיאוריה ו‑UX)

BottomNavigationView הוא רכיב Material שמאפשר ניווט ראשי בין יעדים (destinations) ברמת היישום.

התנהגויות חשובות:
- כל פריט מייצג בדרך כלל יעד (Fragment).
- בחירה בפריט מעבירה את המשתמש ליעד המתאים (במדריך זה — החלפה של ה‑Fragment).
- BottomNavigationView מנהל מצב בחירה, אייקונים וכיתובים.

בשיעור זה נשמור על פשטות:
- נשתמש ב‑replace כאשר נבחר פריט. זה הפתרון הפשוט והקריא למתחילים.
- הערה: באפליקציות מורכבות עדיף לשמור מצבים או להשתמש ב‑Navigation Component.

---

## 6 — תפריט הניווט התחתון

`res/menu/bottom_nav_menu.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/nav_fragment1"
        android:icon="@drawable/ic_looks_one"
        android:title="Fragment 1" />
    <item
        android:id="@+id/nav_fragment2"
        android:icon="@drawable/ic_looks_two"
        android:title="Fragment 2" />
    <item
        android:id="@+id/nav_fragment3"
        android:icon="@drawable/ic_looks_3"
        android:title="Fragment 3" />
</menu>
```

הערות להוראה:
- לצורך נגישות ולתרגום יש להכניס כיתובים ל‑strings.xml; כאן שמרנו על פשטות כדי להקל על הקריאה.
- וודאו שיש לכם drawables מתאימים או השתמשו באייקונים של Material.

---

## 7 — פריסת ה‑Activity הראשית (activity_main.xml)

פריסת LinearLayout אנכית פשוטה עם מכולת ל‑Fragment ו‑BottomNavigationView:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- מכולת פשוטה ל‑Fragment -->
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- ניווט תחתון (Material) -->
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu" />
</LinearLayout>
```

טיפ למורה:
- FrameLayout הוא פשוט להסבר. מאוחר יותר ניתן להראות FragmentContainerView כחלופה מודרנית.

---

## 8 — MainActivity הפשוטה (קוד + הסברים)

`MainActivity.java`
```java
package com.example.bottomnavdemo;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import com.google.android.material.bottomnavigation.BottomNavigationView;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

         BottomNavigationView bottomNav = findViewById(R.id.bottom_navigation);

         // הצגת Fragment1 כברירת מחדל
         if (savedInstanceState == null) {
             getSupportFragmentManager().beginTransaction()
                 .replace(R.id.fragment_container, new Fragment1())
                 .commit();
         }

         bottomNav.setOnItemSelectedListener(item -> {
             Fragment selectedFragment = null;
             switch (item.getItemId()) {
                 case R.id.nav_fragment1:
                     selectedFragment = new Fragment1();
                     break;
                 case R.id.nav_fragment2:
                     selectedFragment = new Fragment2();
                     break;
                 case R.id.nav_fragment3:
                     selectedFragment = new Fragment3();
                     break;
             }
             if (selectedFragment != null) {
                 // החלפה בפשטות: אנו יוצרים מופע חדש של ה‑Fragment בכל בחירה
                 getSupportFragmentManager().beginTransaction()
                     .replace(R.id.fragment_container, selectedFragment)
                     .commit();
             }
             return true;
         });
    }
}
```

הסברים:
- `getSupportFragmentManager()` מחזיר את ה‑FragmentManager שמנהל את הטרנזקציות (add/replace/remove).
- `replace()` מסיר את ה‑Fragment הקיים ומוסיף את החדש — פתרון פשוט ומובן למתחילים.
- הבדיקה `savedInstanceState == null` מונעת החלפת ה‑Fragment שהשחזור (restore) של מערכת ה‑Activity כבר ביצעה.

---

## 9 — פריסות ה‑Fragments (פשוטות)

כל ה‑Fragments משתמשים באותה מבנה בסיסי. העתיקו והתאימו ל‑fragment_2.xml ו‑fragment_3.xml עם IDs מתאימים.

`res/layout/fragment_1.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:gravity="center"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp">

    <Button
        android:id="@+id/btnFragment1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Fragment 1" />

    <TextView
        android:id="@+id/tvFragment1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="18sp"
        android:layout_marginTop="20dp" />
</LinearLayout>
```

שנו ל‑btnFragment2 / tvFragment2 ול‑btnFragment3 / tvFragment3 בקבצים המתאימים.

---

## 10 — קוד ה‑Fragment עם שימוש נכון במחזור החיים

`Fragment1.java`
```java
package com.example.bottomnavdemo;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class Fragment1 extends Fragment {
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        // טוענים ומחזירים את ה‑View של ה‑Fragment. אין לגשת לילדים כאן.
        return inflater.inflate(R.layout.fragment_1, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view,
                              @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        // כאן בטוח לגשת ל‑Views ולהגדיר מאזינים
        Button btn = view.findViewById(R.id.btnFragment1);
        TextView tv = view.findViewById(R.id.tvFragment1);

        btn.setOnClickListener(v -> tv.setText("לחצת על הכפתור ב‑Fragment מספר 1"));
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        // אם שמרת כאן שדות שמצביעים על Views — נקה אותם כדי למנוע דליפות זיכרון.
    }
}
```

טיפים להוראה:
- הדגישו שהתנהגות זו מונעת NullPointerException ושומרת על הפרדה בין יצירת ה‑View להגדרות ה‑UI.

---

## 11 — מה קורה בסיבוב מסך? (הסבר פשוט)

- ברירת המחדל: כאשר מסתובב המכשיר, ה‑Activity וה‑Fragments מתמוטטים ויוצרים מחדש.
- במדריך הזה, מאחר שאנו יוצרים מחדש את ה‑Fragments בכל בחירה, לאחר סיבוב המסך יופיע ה‑Fragment ברירת המחדל (Fragment1) — זה מכוון כדי לשמור על פשטות ההסבר.
- אם רוצים לשמור את הטאב שנבחר, אפשר:
  - לשמור את id הפריט שנבחר ב‑onSaveInstanceState ולהחזיר אותו ב‑onCreate, או
  - להשתמש בדפוס של הוספה/הסתרה (add/hide) או ב‑Navigation Component (נושאים מתקדמים יותר).
- לשם הוראה: התלמידים יכולים להשתמש בחוויה זו כדי להבין למה שימור state חשוב.

דוגמה קצרה לשמירת פריט נבחר:
```java
// במשתנים של ה‑Activity
private static final String SELECTED_ITEM = "selected_item";
private int selectedItemId = R.id.nav_fragment1;

// ב‑onCreate: שיחזור
if (savedInstanceState != null) {
    selectedItemId = savedInstanceState.getInt(SELECTED_ITEM, R.id.nav_fragment1);
    bottomNav.setSelectedItemId(selectedItemId);
}

// בבורר הרשימה:
selectedItemId = item.getItemId();

// ב‑onSaveInstanceState:
outState.putInt(SELECTED_ITEM, selectedItemId);
```

---

## 12 — תקשורת Activity ↔ Fragment (אפשרויות פשוטות)

מקרה נפוץ: ה‑Activity צריך לומר ל‑Fragment משהו או להיפך.

אפשרויות פשוטות למתחילים:
1. קריאה ישירה למתודות של ה‑Fragment דרך FragmentManager:
   - Activity יכול למצוא את ה‑Fragment ולקרוא לשיטה ציבורית שלו.
   - דוגמה: `Fragment1 f = (Fragment1) getSupportFragmentManager().findFragmentById(R.id.fragment_container); if (f != null) f.updateTitle("...");`
   - עובד רק אם יש גישה למופע המתאים (למשל שמרו אותו בתג או בשדה).
2. שימוש בממשק (interface):
   - ה‑Fragment מגדיר ממשק מאזין שה‑Activity מממש; ה‑Fragment קורא למאזין כדי לדווח אירועים.
3. שימוש ב‑ViewModel משותף (מומלץ באפליקציות גדולות):
   - Activity וה‑Fragments צופים ב‑LiveData משותף ב‑ViewModel — פתרון מודרני ומבוסס lifecycle (נושא מתקדם).

במדריך זה נשמור על הפתרון הפשוט: אם צריך Activity→Fragment ניתן לאתחל Fragment עם ארגומנטים או למצוא את המופע ולהתקשר אליו ישירות.

---

## 13 — replace vs add/hide vs back stack (תיאוריה קצרה)

- replace(container, fragment): מסיר ומוסיף את ה‑Fragment החדש. פשוט וברור — לכן בבסיס המדריך אנחנו משתמשים בו.
- add(container, fragment): מוסיף fragment נוסף מעל הקיים (שימושי להצגה מרובה).
- hide(fragment) / show(fragment): שומר על ה‑Fragments בזיכרון ומאפשר לשמור על מצב ה‑UI.
- addToBackStack(name): אם מוסיפים את הטרנזקציה ל‑back stack, לחצן חזרה יבטל את הטרנזקציה. ברוב אפליקציות עם טאבים נמנעים מהוספת החלפות טאבים ל‑back stack.

למה השתמשנו ב‑replace?
- בגלל פשטות ההסבר והקוד.
- החיסרון: מצב ה‑Fragment נאבד כאשר הוא מוחלף — אנחנו מקבלים זאת במודע כדי לשמור על מדריך קצר ונקי.

---

## 14 — נגישות וטיפים קטנים לביצוע טוב

- השתמשו ב‑strings.xml עבור טקסטים כדי לאפשר תרגום ונגישות.
- וודאו שלאייקונים יש משמעות ברורה; הוסיפו contentDescription אם צריך (בעיקר ל־ImageView).
- בדקו עם TalkBack או מצביעי נגישות בסיסיים.
- הימנעו מטקסטים קשים לקריאה (גדלי טקסט קטנים) ובעיות קונטרסט.

---

## 15 — תרגילים פשוטים לתלמידים

1. הריצו את המדריך: לחצו בכל פריט ניווט תחתון, לחצו על הכפתור בכל Fragment וצפו ב‑TextView שמתעדכן.
2. סובבו את המכשיר: מה קורה? זהו הזדמנות להסביר שחזור ולהציג onSaveInstanceState.
3. בנו ספירה מקומית ב‑Fragment1 שמתרעננת בכל לחיצה על הכפתור והציגה אותה. שימו לב ששינוי בין Fragments מאפס את הספירה (עקב יצירת מופעים חדשים).
4. אתגר מתקדם: שמרו את הטאב הנבחר לאחר סיבוב מסך בעזרת onSaveInstanceState (דוגמה בסעיף 11).

---

## 16 — מה השמטנו בכוונה (ולמה)

לשמירה על פשטות:
- לא השתמשנו ב‑ViewBinding — מצמצם מעט את ההסבר למתחילים.
- לא השתמשנו ב‑add/hide או ב‑Navigation Component — נושאים אלו מוסיפים מורכבות של מחזור חיים/Back Stack.
- לא נכנסו ל‑LiveData/ ViewModel — שיעור נפרד יתאים לזה.

---

## 17 — סיכום

מדריך זה שומר על פשטות כדי שהתלמידים יתמקדו ב:
- מהו Fragment
- ההבדל בין onCreateView ל‑onViewCreated
- איך להחליף Fragments באמצעות BottomNavigationView ו‑FragmentManager

הסברים התיאורטיים שמצוינים כאן יעזרו להם להבין לא רק את ה"איך" אלא גם את ה"למה", ויתנו בסיס לנושאים מתקדמים יותר: שימור מצב, Navigation Component, ViewBinding ו‑ViewModel.

---

אם תרצו, אני יכול:
- לבצע commit של קובץ Markdown זה לריפוזיטורי שלכם, או
- להוסיף גם את קבצי ה‑Java וה‑XML המלאים מוכנים להדבקה ב‑Android Studio ולבצע commit.

איזה מהאפשרויות להמשיך כעת? (אישית אני ממליץ להוסיף גם את הקבצים המוכנים כדי שהסטודנטים יוכלו להעתיק ולהריץ במהירות).
