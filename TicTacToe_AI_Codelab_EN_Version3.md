# Codelab: Creating AI Players for Tic Tac Toe in Android (Java, Beginner Level)

Welcome! In this extension, you'll learn how to add **computer (AI) players** to your Tic Tac Toe game.  
You'll start by defining a clear way to represent moves, understand what a "computer" player is, and build progressively smarter AI:  
- **Random AI** (picks any move)
- **Heuristic AI** (plays more intelligently)

We'll also discuss what "heuristics" are, why they're useful, and introduce you to the basics of AI and its evolution.

---

## 1. Introducing the Move Class

Before building computer players, let's create a simple class to represent a move.

### **Why use a Move class?**
- Makes your code easier to read and maintain.
- Lets you add more information later (like a score for AI).
- Avoids confusion with arrays or multiple variables.

### **Add this to your project:**

```java
public class Move {
    public int row;
    public int col;

    public Move(int row, int col) {
        this.row = row;
        this.col = col;
    }
}
```

**Now, whenever you want to represent a move, use `Move` instead of `int[]` or two separate variables.**

---

## 2. What is a "Computer" Player?

In Tic Tac Toe, a "computer" player is a program that decides where to place its mark (X or O) automatically, instead of a human clicking a button.

- **Why use a computer player?**
    - Practice: Lets you play against the app.
    - Challenge: Can be made as easy or hard as you want.
    - Learning: Demonstrates how simple logic can lead to smart behavior.

A computer player is just a function (or method) that, given the current board, returns a move for the computer.

---

## 3. Theory: Getting All Possible Moves

Before the computer can choose a move, it needs to know **which moves are legal** (empty cells).

**Theory:**  
- The Tic Tac Toe board is a 3x3 grid.
- Each cell is either empty, or has X or O.
- A "possible move" is any empty cell.

**Code Example:**

```java
public ArrayList<Move> getPossibleMoves() {
    ArrayList<Move> moves = new ArrayList<>();
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (board[i][j] == EMPTY) {
                moves.add(new Move(i, j));
            }
        }
    }
    return moves;
}
```

**Explanation:**  
- This method scans the board and adds each empty cell as a `Move` to a list.
- The computer player will use this list to decide its move.

---

## 4. The Simplest Computer: Random AI

**Theory:**  
- The easiest way for a computer to play is to **pick any possible move at random**.
- This AI has no strategy, but it demonstrates how a computer player works.

**Benefits:**
- Easy to implement.
- Great for testing your game.
- Not challenging (easy to beat!).

**Code Example:**

```java
public Move getRandomMove() {
    ArrayList<Move> moves = getPossibleMoves();
    if (moves.isEmpty()) return null; // No moves left
    Random rand = new Random();
    return moves.get(rand.nextInt(moves.size()));
}
```

**How to use:**  
After the human plays, call `getRandomMove()` for the computer, and make that move on the board.

---

## 5. What Are Heuristics (and Why Are They Needed)?

### **Definition:**
A **heuristic** is a rule or method that helps the computer make decisions more intelligently, often based on patterns or experience, rather than brute force.

### **Why use heuristics in games?**
- **Random play is weak:** The computer misses easy wins and fails to block you.
- **Heuristics make the computer smarter:**  
    - Wins when it can.
    - Blocks you from winning.
    - Takes better positions (like the center).

### **Examples of heuristics in Tic Tac Toe:**
- If I can win, do it.
- If the opponent can win next turn, block it.
- Take the center if it's free.
- Take a corner if possible.

**Good heuristics:**  
- Help the computer win or tie more often.
- Make the game fun and challenging.

**Bad heuristics:**  
- Ignore obvious threats.
- Always pick the same square.
- Don't adapt to the situation.

---

## 6. Building a Heuristic AI for Tic Tac Toe

**Theory:**  
A simple heuristic AI in Tic Tac Toe can:
1. Win if possible.
2. Block the opponent's win.
3. Take center if available.
4. Take a corner if available.
5. Otherwise, pick random.

**Code Example:**

```java
public Move getHeuristicMove(int aiPlayer, int humanPlayer) {
    // 1. Win if possible
    for (Move move : getPossibleMoves()) {
        board[move.row][move.col] = aiPlayer;
        if (checkWin() == aiPlayer) {
            board[move.row][move.col] = EMPTY;
            return move;
        }
        board[move.row][move.col] = EMPTY;
    }
    // 2. Block if needed
    for (Move move : getPossibleMoves()) {
        board[move.row][move.col] = humanPlayer;
        if (checkWin() == humanPlayer) {
            board[move.row][move.col] = EMPTY;
            return move;
        }
        board[move.row][move.col] = EMPTY;
    }
    // 3. Take center
    if (board[1][1] == EMPTY)
        return new Move(1, 1);

    // 4. Take a corner
    int[][] corners = {{0,0}, {0,2}, {2,0}, {2,2}};
    for (int[] c : corners)
        if (board[c[0]][c[1]] == EMPTY)
            return new Move(c[0], c[1]);

    // 5. Otherwise, pick random
    return getRandomMove();
}
```

**Discussion:**  
- This AI will never lose to random, and will rarely lose to a human.

---

## 7. The Origins of AI (Brief History)

**Artificial Intelligence (AI)** began as a field in the 1950s, with the goal of making machines that "think" like humans.

- **Early AI:** Played games like chess and checkers, using simple rules and brute-force search.
- **Modern AI:** Uses advanced algorithms, learning from data (machine learning, deep learning).
- **In games:** AI has gone from simple heuristics to unbeatable computers (e.g., Deep Blue in chess, AlphaZero in board games).

**In Tic Tac Toe:**  
- AI can be random, use heuristics, or search all possible moves to play perfectly.
- You are now building the foundation for understanding how game-playing AI works!

---

## 8. Summary and Next Steps

- You learned how to represent moves with a `Move` class.
- Built a random computer player.
- Learned about heuristics and built a smarter AI.
- Discovered what AI is and its roots in game playing.

**Next:**  
Try playing against your computer player.  
Experiment with your own heuristics!  
Later, you can learn about search algorithms like Negamax/Minimax for unbeatable AI.

---

*Happy coding!*