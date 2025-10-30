# Android Codelab — Single Activity, Three Fragments, Bottom Navigation (Keep It Simple)

This codelab teaches fragments and bottom navigation in the simplest possible way for beginners:
- One Activity (MainActivity)
- Three Fragments (Fragment1, Fragment2, Fragment3)
- Bottom navigation to switch between fragments
- Each fragment has a Button and a TextView; pressing the button updates its TextView
- All view lookup is done with findViewById (no ViewBinding)

Focus: basic fragment lifecycle (onCreateView vs onViewCreated) and how to swap fragments with the FragmentManager. We keep the UX and code minimal for teaching, while adding explanation of the underlying concepts.

---

## Quick overview — what you'll learn and why

- What a Fragment is and how it relates to an Activity
- The fragment view lifecycle (why inflate in onCreateView and set up UI in onViewCreated)
- How the FragmentManager swaps fragments (replace vs add/hide)
- How BottomNavigationView works and how to handle user navigation simply
- Basic state concerns: what happens on rotation and how to keep example simple
- Simple Activity↔Fragment communication options
- Accessibility and small best-practices for beginners

---

## 1 — Add material dependency

In your module `build.gradle` (Module: app) add the material library if not already present:

```gradle
dependencies {
    implementation (libs.material)
    // ... other dependencies
}
```

Sync the project.

---

## 2 — Project structure (simple)

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

## 3 — What is a Fragment? (theory)

A Fragment is a reusable piece of UI with its own lifecycle that an Activity can host. Think of fragments as modular screens you can plug into activities. They are not independent apps — a fragment always runs inside an activity.

Why use fragments?
- Reuse UI components across different screen sizes (phone vs tablet).
- Build flexible UIs (swap parts of the screen without replacing the whole Activity).
- Keep screen-specific logic modular (each fragment handles its own UI).

Key idea: fragments have both an *own lifecycle* and a *view lifecycle*. The view lifecycle is important because the fragment object can outlive its view (e.g., when the view is destroyed during configuration changes but the fragment instance is reattached).

---

## 4 — Fragment view lifecycle (important to understand)

Relevant fragment lifecycle methods for this codelab:

- onAttach() — fragment is attached to Activity (rarely used for simple UI wiring)
- onCreate() — fragment instance is being created (useful for retained non-view state)
- onCreateView() — inflate and return the view hierarchy for the fragment
  - Inflate the layout here. Do not access child views (some students try and see nulls).
- onViewCreated() — called after onCreateView; the view hierarchy exists and you can safely find views with findViewById and set listeners here
  - Good place to set adapters, listeners, and populate UI controls
- onStart(), onResume() — fragment is visible and interacting with the user
- onPause(), onStop() — fragment visible state changes
- onDestroyView() — fragment's view is destroyed (null out view references here to avoid leaks)
- onDestroy() — fragment instance is destroyed

Important teaching points:
- Do not keep direct references to Views in fragment fields longer than the view lifetime — clear them in onDestroyView().
- onCreateView returns the inflated View: returning a non-null View is required.

---

## 5 — Bottom navigation basics (theory and UX)

BottomNavigationView is a Material component that offers persistent, primary navigation between top-level destinations.

Important behaviors:
- Each item typically corresponds to a top-level destination (a fragment in our codelab).
- Selecting an item should navigate to that destination (in this codelab we replace the fragment).
- BottomNavigation handles selection state and icons/titles automatically.

Keep it simple:
- We replace fragments on selection. This is easy to explain and read for beginners.
- Note: In real apps you might preserve fragment instances or use the Navigation Component to maintain back stacks. We'll note those options later.

---

## 6 — Bottom navigation menu

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

Teaching notes:
- For accessibility and localization, put titles in `strings.xml`. This example keeps it simple for readability.
- Choose simple, meaningful icons and ensure they are included as drawables or use Material icons.

---

## 7 — Main layout (activity_main.xml)

Use a simple vertical LinearLayout with a FrameLayout fragment host and BottomNavigationView.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- Simple fragment host -->
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- Bottom navigation (Material) -->
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu" />
</LinearLayout>
```

Teacher tip:
- FrameLayout is simple to explain as a container. If you later want to introduce FragmentContainerView you can swap it in, but FrameLayout is clear for beginners.

---

## 8 — The simplest MainActivity (code + explanations)

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

         // Show Fragment1 by default
         if (savedInstanceState == null) {
             getSupportFragmentManager().beginTransaction()
                 .replace(R.id.fragment_container, new Fragment1())
                 .commit();
         }

         bottomNav.setOnItemSelectedListener(item -> {
            Fragment selectedFragment = null;
            if (item.getItemId()==R.id.nav_fragment1) {

                selectedFragment = new Fragment_1();
            }
            else  if (item.getItemId()== R.id.nav_fragment2) {
                selectedFragment = new Fragment_2();
            }
            else
            {
                    selectedFragment = new Fragment_1();

            }
             if (selectedFragment != null) {
                 // Replace the fragment in the container.
                 // This creates a new fragment instance each time for simplicity.
                 getSupportFragmentManager().beginTransaction()
                     .replace(R.id.fragment_container, selectedFragment)
                     .commit();
             }
             return true;
         });
    }
}
```

Explanations:
- `getSupportFragmentManager()` gives the FragmentManager that manages fragment transactions (add/replace/remove).
- `replace()` removes any existing fragment in the container and adds the new one. It is simple and readable.
- We only call replace when the user selects a different tab.
- We only add the default fragment when savedInstanceState == null — this prevents overwriting the currently restored fragment on rotation.

---

## 9 — Fragment layouts (simple)

All three fragments use the same simple layout structure. Copy and adjust IDs for fragment_2.xml and fragment_3.xml.

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

Repeat for fragment_2.xml and fragment_3.xml with IDs changed to `btnFragment2` etc.

---

## 10 — Fragment code with clear lifecycle usage

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
        // Inflate and return the fragment's view. Do not access child views here.
        return inflater.inflate(R.layout.fragment_1, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view,
                              @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        // Now it's safe to access child views and set listeners
        Button btn = view.findViewById(R.id.btnFragment1);
        TextView tv = view.findViewById(R.id.tvFragment1);

        btn.setOnClickListener(v -> tv.setText("Clicked button in fragment number 1"));
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        // If you had stored view references in fields, null them here to avoid leaks.
    }
}
```

Teaching tips:
- Explain that the fragment object may survive while its view is destroyed (e.g., orientation change). That is why view references should be cleared in `onDestroyView()`.

---

## 11 — What happens on rotation? (simple explanation)

- When the device rotates, by default the Activity and its fragments are destroyed and recreated.
- Because our codelab recreates fragments on selection, after a rotation the app will re-create the default fragment (Fragment1) unless you handle savedInstanceState or preserve selected item.
- To keep the current selected tab on rotation you can:
  - Save the selected menu id in `onSaveInstanceState` and restore it in `onCreate`, or
  - Use a caching approach (add/hide fragments and restore them) or
  - Use the Navigation Component (advanced).
- For this beginner codelab, we intentionally keep the behavior simple: students will see the fragments recreated — a chance to demonstrate what "stateless" code looks like and to motivate the next lesson on preserving state.

Example: saving selected item (brief)
```java
// in MainActivity fields
private static final String SELECTED_ITEM = "selected_item";
private int selectedItemId = R.id.nav_fragment1;

// in onCreate: restore if present
if (savedInstanceState != null) {
    selectedItemId = savedInstanceState.getInt(SELECTED_ITEM, R.id.nav_fragment1);
    bottomNav.setSelectedItemId(selectedItemId);
}

// when nav item selected:
selectedItemId = item.getItemId();

// in onSaveInstanceState:
outState.putInt(SELECTED_ITEM, selectedItemId);
```

---

## 12 — Activity ↔ Fragment communication (simple options)

Common requirement: the Activity needs to tell a fragment something or the fragment needs to notify the Activity.

Simple approaches for beginners:
1. Call fragment methods via FragmentManager:
   - Activity can find a visible fragment and call a public method on it.
   - Example: `Fragment1 f = (Fragment1) getSupportFragmentManager().findFragmentById(R.id.fragment_container); if (f != null) f.updateTitle("...");`
   - Only works if you keep a reference or can find the fragment tag.

2. Use interfaces (classic pattern):
   - Fragment defines a listener interface that the Activity implements.
   - Fragment calls the listener method to notify the Activity.

3. Use a shared ViewModel (recommended for larger apps):
   - Both Activity and Fragments observe LiveData in a shared ViewModel. This decouples components and is lifecycle-aware (advanced topic).

For the codelab keep it simple: if you need Activity→Fragment communication, the Activity can recreate the fragment with arguments or find the fragment instance using a tag and call a method on it. We'll cover ViewModel-based communication in later lessons.

---

## 13 — Replace vs add/hide vs back stack (short theory)

- replace(container, fragment): removes any existing fragment in the container and adds the new one. Good for simple tab switching example.
- add(container, fragment): adds a fragment on top; previous fragments remain attached (useful when you want to show multiple fragments at once).
- hide(fragment) / show(fragment): lets you keep fragments in memory and preserve their UI state.
- addToBackStack(name): if you add the transaction to the back stack, pressing back will reverse the transaction. For bottom navigation, apps often avoid adding tab switches to the back stack.

Why we use replace() here:
- Simplicity: easy to explain, predictable behavior for beginners.
- No back stack complexity when switching tabs.
- The trade-off: fragment state is lost when replaced — we accept that here to keep the lesson focused.

---

## 14 — Accessibility & small best practices

- Use string resources (strings.xml) for UI text so the app can be localized.
- Ensure icons are meaningful; provide `android:contentDescription` where appropriate (especially for ImageViews).
- Test content with TalkBack if possible — bottom navigation works well with proper labels.
- Avoid hard-coded text in production code; keep the codelab simple for clarity.

---

## 15 — Simple exercises for students

1. Try the codelab: run it, click each bottom nav item, press the buttons, and observe the TextView updates.
2. Rotate the device: what happens to the visible fragment? Use this to explain recreation and savedInstanceState.
3. Modify Fragment1 to increment a local counter each time the button is pressed, and display the counter. Notice that switching fragments resets the counter (because fragments are recreated in this simple design).
4. Advanced: preserve the selected tab on rotation using `onSaveInstanceState` (short exercise shown earlier).

---

## 16 — What we intentionally left out (and why)

To keep the lesson short and beginner-friendly we avoided:
- ViewBinding — reduces boilerplate but adds an extra concept. We can introduce it later.
- Fragment caching (add/hide) or the Navigation Component — both introduce additional concepts (fragment lifecycle complexity and back stack management). We'll teach them in follow-up lessons.
- Complex state management patterns (LiveData, ViewModel) — left for an intermediate lesson.

---

## 17 — Summary

This minimal codelab uses findViewById and simple replace transactions so beginners can focus on:
- What a fragment is
- The difference between onCreateView and onViewCreated
- How to swap fragments using BottomNavigationView and FragmentManager

By adding the theoretical notes above, students will understand not only the "how" but also the "why" — preparing them to progress to more advanced topics: preserving fragment state, Navigation Component, ViewBinding and ViewModel communication.

---

If you'd like, I can:
- Commit this "Keep It Simple" codelab markdown to your repository, or
- Also add the ready-to-paste Java + XML files for the minimal project and commit them.

Which would you prefer? (I can commit files now if you want.)
