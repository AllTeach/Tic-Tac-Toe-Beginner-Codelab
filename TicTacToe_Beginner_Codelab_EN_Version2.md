# Android Codelab: Building a Tic Tac Toe Game (Java, Beginner Level)

Welcome! In this codelab, you'll create a simple **Tic Tac Toe** game for Android using Java. You'll apply the **MVC (Model-View-Controller)** pattern, where:

- The **View** is your layout XML (only displays, no logic).
- The **Model** is a Java class handling game logic (board, moves, win/tie checks).
- The **Controller** is your `Activity`, connecting user actions to the model and updating the view.

> **Level:** Beginners  
> **Assumptions:** You know Java basics, inheritance, and basic Android XML (buttons, image views, drawables, `android:onClick`), but not interfaces or polymorphism.

---

## Table of Contents

1. Introduction to MVC and Project Overview
2. Step 1: Designing the Layout (View-only)
    - Why Use GridLayout?
3. Step 2: Handling User Clicks
4. Step 3: Identifying Button Location (row, col) using Tag
5. Step 4: Building the Model Class (Game Logic)
6. Step 5: Connecting Model and Controller (Activity)
7. Further Reading
8. Summary

---

## Introduction to MVC and Project Overview

**MVC Pattern Recap**
- **Model:** Handles data and logic (game board, win/tie logic).
- **View:** The XML layout (what the user sees; buttons/imageViews).
- **Controller:** The Activity (receives clicks, calls model, updates UI).

MVC helps keep code clean and maintainable by separating concerns.

**Goal:**  
Create a Tic Tac Toe game where the UI reacts to user clicks, the model manages the game state, and the activity acts as the bridge.

---

## Step 1: Designing the Layout

### What You'll Learn
- Designing a grid with buttons or imageViews for Tic Tac Toe.
- How to use drawables for X and O images.
- Making the layout “view-only” (no logic inside XML).

### 1.1 Create a New Android Studio Project

- **Open Android Studio and choose “New Project”.**
- **Select “Empty Views Activity”** (recent Android Studio versions).
- **Programming Language:** Choose **Java**.
- **Minimum SDK:** Set to **API 30 (Android 11.0)** or higher.

This sets up a simple activity where you can add your GridLayout and game logic.

---

### Why Use GridLayout?

So far, you may have used **ConstraintLayout** for arranging elements. ConstraintLayout is powerful and flexible for complex UIs, but for simple grid-like arrangements (like a Tic Tac Toe board), **GridLayout** is much easier and cleaner.

**What is GridLayout?**
- GridLayout arranges its child views in a rectangular grid (rows and columns).
- You specify how many rows and columns you want (`android:rowCount`, `android:columnCount`), and then add your child views (Buttons, ImageViews) in order.
- Each view takes a cell in the grid, and you can easily control their positions.

**Why Choose GridLayout for Tic Tac Toe?**
- Tic Tac Toe is naturally a 3x3 grid.
- GridLayout lets you express this directly in XML, making your code simpler and easier to understand.
- No need for complex constraints or manual positioning—just set the grid size and add your buttons!

**Comparison: ConstraintLayout vs GridLayout**
- **ConstraintLayout:** Good for flexible, complex layouts; but positioning a 3x3 grid requires many constraints.
- **GridLayout:** Made for grid arrangements; perfect for games like Tic Tac Toe!

**Official Reference:**  
- [GridLayout Documentation](https://developer.android.com/reference/android/widget/GridLayout)
- [GridLayout vs ConstraintLayout Stack Overflow](https://stackoverflow.com/questions/41649494/when-to-use-gridlayout-vs-constraintlayout)

---

### 1.2 The Layout File (`activity_main.xml`)
We'll use a **GridLayout** for the board:

```xml
<GridLayout
    android:id="@+id/gridLayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:rowCount="3"
    android:columnCount="3"
    android:layout_gravity="center"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- 9 buttons for the board -->
    <Button
        android:id="@+id/btn00"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:tag="00"
        android:onClick="onCellClick"
        android:text="" />

    <Button
        android:id="@+id/btn01"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:tag="01"
        android:onClick="onCellClick"
        android:text="" />
    <!-- ... repeat for all 9 buttons (tags: 00, 01, 02, 10, 11, 12, 20, 21, 22) -->
</GridLayout>
```
**Notes:**
- Each button uses `android:tag` to store its row and column (e.g., "01" = row 0, col 1).
- Use `android:onClick` to call a Java method when clicked.

**Download X and O images**
- Place them in `res/drawable` (e.g., `x.png`, `o.png`).
- [How to Add Images](https://developer.android.com/studio/write/image-asset-studio)

---

## Step 2: Handling User Clicks

### What You'll Learn
- How `android:onClick` works.
- How to handle a button click in your Activity.

**In your `MainActivity.java`:**

```java
public void onCellClick(View view) {
    // This method is called when any board button is clicked
    // We'll handle extracting row/col and calling the model later
}
```
- The method receives the clicked `View` (in this case, a Button).
- All board buttons call this same method.

[More on Button Clicks](https://developer.android.com/training/basics/firstapp/starting-activity)

---

## Step 3: Identifying Button Location (row, col) using Tag

### Why Use Tag?
- The tag lets you associate data (row, col) with each button.
- Easier and less error-prone than parsing IDs.

### How to Extract Row and Column

```java
public void onCellClick(View view) {
    String tag = (String) view.getTag(); // e.g. "12"
    int row = Character.getNumericValue(tag.charAt(0));
    int col = Character.getNumericValue(tag.charAt(1));
    // Pass row and col to the model
}
```

**Tips:**
- You can use `android:tag` for any custom data.
- [View Tags Documentation](https://developer.android.com/reference/android/view/View#setTag(java.lang.Object))

---

## Step 4: Building the Model Class (Game Logic)

### What You'll Learn
- How to design a model class for game logic.
- Encapsulation, state, and methods.

### 4.1 Define the API

**Create a `TicTacToeModel.java`:**
```java
public class TicTacToeModel {
    public static final int EMPTY = 0;
    public static final int PLAYER_X = 1;
    public static final int PLAYER_O = 2;

    private int[][] board;
    private int currentPlayer;

    public TicTacToeModel() {
        board = new int[3][3];
        currentPlayer = PLAYER_X;
    }

    public int getCurrentPlayer() {
        return currentPlayer;
    }

    public boolean isLegal(int row, int col) {
        return board[row][col] == EMPTY;
    }

    public boolean makeMove(int row, int col) {
        if (!isLegal(row, col)) return false;
        board[row][col] = currentPlayer;
        return true;
    }

    public void changePlayer() {
        currentPlayer = (currentPlayer == PLAYER_X) ? PLAYER_O : PLAYER_X;
    }

    public int checkWin() {
        // Returns PLAYER_X, PLAYER_O, or EMPTY (no winner)
        // Implement row, col, diagonal checks
        // (see below for example)
        return EMPTY;
    }

    public boolean isTie() {
        // If all cells filled and no winner
        return false;
    }

    public void resetGame() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                board[i][j] = EMPTY;
        currentPlayer = PLAYER_X;
    }
}
```
**You’ll need to implement `checkWin` and `isTie`.**  
- [Arrays in Java](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html)
- [Encapsulation](https://developer.android.com/training/basics/firstapp/starting-activity)

### 4.2 Reasoning
- Use constants for clarity (`PLAYER_X`, `PLAYER_O`).
- Encapsulate the board and current player in the model.
- Methods ensure only valid moves, switching turns, and game status.

---

## Step 5: Connecting Model and Controller (Activity)

### What You'll Learn
- How to create a model instance in your Activity.
- How to use the model after user clicks.

**In your `MainActivity.java`:**

```java
public class MainActivity extends AppCompatActivity {
    private TicTacToeModel model;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        model = new TicTacToeModel();
    }

    public void onCellClick(View view) {
        String tag = (String) view.getTag();
        int row = Character.getNumericValue(tag.charAt(0));
        int col = Character.getNumericValue(tag.charAt(1));

        if (model.isLegal(row, col)) {
            model.makeMove(row, col);
            // Update UI: set X or O image/text on button
            Button btn = (Button) view;
            if (model.getCurrentPlayer() == TicTacToeModel.PLAYER_X) {
                btn.setBackgroundResource(R.drawable.x); // or setText("X")
            } else {
                btn.setBackgroundResource(R.drawable.o); // or setText("O")
            }

            if (model.checkWin() != TicTacToeModel.EMPTY) {
                // Show winner (Toast, Dialog, etc.)
            } else if (model.isTie()) {
                // Show tie
            } else {
                model.changePlayer();
            }
        } else {
            // Illegal move (cell already taken)
        }
    }
}
```

**Explanations:**
- The Activity holds an instance of the model.
- On click, row/col are extracted and passed to the model.
- The view is updated according to model state.

[How to Update UI](https://developer.android.com/guide/topics/ui/controls/button)

---

## Further Reading & Official Links

- [Android Layouts Guide](https://developer.android.com/guide/topics/ui/declaring-layout)
- [View Tags](https://developer.android.com/reference/android/view/View#setTag(java.lang.Object))
- [GridLayout Documentation](https://developer.android.com/reference/android/widget/GridLayout)
- [MVC Pattern in Android](https://developer.android.com/guide/topics/architecture)
- [Toast Messages](https://developer.android.com/guide/topics/ui/notifiers/toasts)

---

## Summary

You’ve built a simple Tic Tac Toe game in Android using Java, following the MVC pattern.  
**Key steps:**
- Created a grid layout with clickable buttons (View).
- Used `android:tag` to store row/col data.
- Built a model class encapsulating game logic (Model).
- Implemented the Activity to connect user actions to the model (Controller).

**Next Steps:**
- Try adding a restart button.
- Improve the UI with images and colors.
- Explore saving game state on rotation.
- Learn about interfaces and advanced patterns!

---

*Happy coding!*