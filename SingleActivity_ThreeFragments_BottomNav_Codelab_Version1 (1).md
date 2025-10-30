# Android Codelab: Single Activity, Three Fragments, Bottom Navigation

---

## **Overview**

This tutorial will guide you through creating an Android project with:
- **One Activity** (`MainActivity`)
- **Three Fragments** (`Fragment1`, `Fragment2`, `Fragment3`)
- **Bottom Navigation** to switch between fragments
- Each fragment with a **Button** and a **TextView**
- When you click the button, the TextView updates with a message specific to the fragment

**The focus:**
- Understanding fragment lifecycle methods: `onCreateView` vs. `onViewCreated`
- Using the FragmentManager to swap fragments
- Implementing bottom navigation

---

## **1. Project Structure**

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
│   └─ drawable/
```

---

## **2. Add Material Library**

In your `build.gradle` (Module: app):

```gradle
    implementation (libs.material)
```

---

## **3. Bottom Navigation Menu**

`res/menu/bottom_nav_menu.xml`
```xml
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
*You can use built-in icons or your own drawables for demonstration.*

---

## **4. Main Layout**

`res/layout/activity_main.xml`
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu"/>
</LinearLayout>
```

---

## **5. Fragment Layouts**

**`res/layout/fragment_1.xml`**
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:gravity="center"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

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

*(Copy & adapt for `fragment_2.xml` and `fragment_3.xml`, changing IDs and Button text to `btnFragment2`, `tvFragment2`, etc.)*

---

## **6. MainActivity.java**

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

        // Show Fragment1 by default
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
                getSupportFragmentManager().beginTransaction()
                    .replace(R.id.fragment_container, selectedFragment)
                    .commit();
            }
            return true;
        });
    }
}
```

---

## **7. Fragment1.java**

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

    // 1. onCreateView: Inflate and return the layout for the fragment
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        // Only inflate and return the view here
        return inflater.inflate(R.layout.fragment_1, container, false);
    }

    // 2. onViewCreated: View is created, set up listeners and UI logic here
    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        Button btn = view.findViewById(R.id.btnFragment1);
        TextView tv = view.findViewById(R.id.tvFragment1);

        btn.setOnClickListener(v -> tv.setText("Clicked button in fragment number 1"));
    }
}
```

---

## **8. Fragment2.java and Fragment3.java**

*(Repeat similar structure, updating IDs and text for each fragment)*

**Fragment2.java**
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

public class Fragment2 extends Fragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_2, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        Button btn = view.findViewById(R.id.btnFragment2);
        TextView tv = view.findViewById(R.id.tvFragment2);

        btn.setOnClickListener(v -> tv.setText("Clicked button in fragment number 2"));
    }
}
```

**Fragment3.java**
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

public class Fragment3 extends Fragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_3, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        Button btn = view.findViewById(R.id.btnFragment3);
        TextView tv = view.findViewById(R.id.tvFragment3);

        btn.setOnClickListener(v -> tv.setText("Clicked button in fragment number 3"));
    }
}
```

---

## **9. Key Learnings: onCreateView vs. onViewCreated**

- **onCreateView:**  
  - Inflate and return the fragment's view (do NOT do logic or access UI elements here)
  - Use only for layout inflation

- **onViewCreated:**  
  - Called **after** the view is created and attached
  - Safe place to find views with `view.findViewById`, set up listeners, adapters, update UI, etc.
  - Keeps code clean and avoids null pointer exceptions

---

## **10. Run & Test**

- Launch the app.
- Tap each bottom navigation item to switch fragments.
- Each fragment should show its own button.
- When you tap the button, the TextView updates with the relevant message.

---

## **Summary Table**

| Fragment Method      | Use For                                  | Example In This Project         |
|----------------------|-------------------------------------------|---------------------------------|
| `onCreateView`       | Inflate the layout, return View           | `inflater.inflate(..., ...)`   |
| `onViewCreated`      | Set up UI logic/listeners on the view     | Set button click listeners     |

---

**You now have a clean demonstration of:**
- Bottom navigation
- FragmentManager usage for swapping fragments
- Fragment lifecycle: onCreateView vs. onViewCreated

---