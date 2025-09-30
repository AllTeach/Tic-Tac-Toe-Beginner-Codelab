# Tic Tac Toe ADVANCED: Unbeatable AI with Negamax & Game Mode Switch

## 1. Adding a Game Mode Switch

### What is a Switch Element?

A **Switch** is a UI component that lets users toggle between two choices (like ON/OFF, TRUE/FALSE). In our app, it will let players choose between **Player vs Player** and **Player vs Computer**.

**Example XML:**
```xml
<Switch
    android:id="@+id/gameModeSwitch"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textOn="Player vs Computer"
    android:textOff="Player vs Player"
    android:checked="false" />
```

**How to Check Switch State in Java:**
```java
Switch gameModeSwitch = findViewById(R.id.gameModeSwitch);
boolean isPvC = gameModeSwitch.isChecked(); // true if Player vs Computer
```
- [Switch documentation](https://developer.android.com/reference/android/widget/Switch)

---

## 2. Implementing Negamax AI

### What is Negamax?

Negamax is a recursive algorithm for zero-sum games (e.g., Tic Tac Toe) that simplifies Minimax by always maximizing the score for the current player.

#### Why Recursion with Negative?

- In Minimax, you alternate between maximizing (for you) and minimizing (for your opponent).
- Negamax simplifies this: every time you recursively call, you "flip" the perspective, so the return value is from the opponent’s point of view.
- We multiply the recursive result by -1 to invert the score (what’s good for your opponent is bad for you).

**Negamax Pseudocode Explained:**
```java
public int negamax(int depth, int player) {
    int winner = checkWin();
    // Stop condition: winner found or board full
    if (winner == player) return +1; // current player wins
    if (winner != EMPTY && winner != player) return -1; // opponent wins
    if (isTie() || depth == 9) return 0; // tie or board full

    int maxScore = Integer.MIN_VALUE;
    for (each empty cell) {
        makeMove(row, col, player);     // simulate move for current player
        int score = -negamax(depth + 1, switchPlayer(player));
        // negate because we're now looking at it from opponent's perspective
        undoMove(row, col);             // backtrack
        if (score > maxScore) {
            maxScore = score;
            bestMove = (row, col);      // keep track of best move
        }
    }
    return maxScore;
}
```
- `makeMove` simulates a possible move.
- We call negamax recursively, switching player (`switchPlayer(player)`).
- The score is negated (`-negamax(...)`) because what’s good for the opponent is bad for us.
- After checking all moves, we return the best score.
- **Notice:** We use `depth == 9` as a stop condition since the board is 3x3 and will be full after 9 moves.

**Why Depth 9 is Unbeatable?**
- In Tic Tac Toe, the deepest possible game is 9 moves.
- Negamax explores every possible outcome, guaranteeing the best result.

---

## 3. Updating the Board for Computer Move

### GridLayout: Direct Mapping vs. Iteration

#### Direct Mapping (Fastest)
If your buttons are added row by row (from top left to bottom right), you can use:
```java
int index = row * 3 + col;
Button btn = (Button) grid.getChildAt(index);
```
This is fast and reliable if you never change the button order.

#### Using findViewWithTag (Recommended for Flexibility)
You can assign each button a tag (e.g., `"01"` for row 0, col 1) and then:
```java
String tag = "" + row + col;
Button btn = grid.findViewWithTag(tag);
```
This works even if you rearrange buttons or add other views.

#### Why Iterate?
Sometimes, you may not know the order or want to be robust against layout changes, so you iterate to find the matching tag:
```java
for (int i = 0; i < grid.getChildCount(); i++) {
    View cell = grid.getChildAt(i);
    if (tag.equals(cell.getTag())) {
        Button btn = (Button) cell;
        // update btn
    }
}
```
But if you use tags and `findViewWithTag`, iteration is not necessary!

---

## 4. Putting It All Together

- Add a Switch for mode selection.
- Implement Negamax in your model.
- When the computer plays, get the move from the model, then update the UI using either direct mapping or `findViewWithTag`.

---

## 5. Challenge & Further Reading

- Try changing depth or making the AI play randomly to see what happens!
- [Negamax](https://en.wikipedia.org/wiki/Negamax)
- [Switch widget](https://developer.android.com/reference/android/widget/Switch)
- [GridLayout](https://developer.android.com/reference/android/widget/GridLayout)

---

*Now your Tic Tac Toe is smarter and more interactive!*