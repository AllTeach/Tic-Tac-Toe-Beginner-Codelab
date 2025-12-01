# Tic‑Tac‑Toe + Firestore — Register once, StartActivity handles registration (student tutorial)

What this tutorial shows (short)
- You register only once (email/password). Registration and the first-time Firestore user document creation happen in StartActivity.
- On later launches StartActivity detects the authenticated user and immediately moves to NamesActivity.
- NamesActivity shows the host name (read from `users/{uid]}`) and has a single EditText for the opponent name.
- After the game finishes we update the host's counters in `users/{uid}` and save a game record into the `Results` collection (host uid, opponent name, result).
- The opponent has no uid — only a name.

Prerequisites (you must already know)
- You know how to authenticate using email and password (createUserWithEmailAndPassword / signInWithEmailAndPassword).
- You know how to access Firestore and how to perform add / set / get operations.

Quick file map (where to put things)
- POJOs: `app/src/main/java/com/example/tictactoe/firebase/`
  - `User.java`, `GameResult.java`
- `StartActivity` (registration — only activity that does sign-up)
- `NamesActivity` (shows host name, enter opponent)
- `GameActivity` / `FinishActivity` (game flow, save results)
- Optional: `HistoryActivity` to list the host's games

---

## 1) POJOs you will use

User.java (saved to `users/{uid}` using `.set(user)`)
```java
// package com.example.tictactoe.firebase;
public class User {
    private String uid;
    private String displayName;
    private String email;
    private int wins;
    private int losses;
    private int draws;

    public User() {} // Firestore requires an empty constructor

    public User(String uid, String displayName, String email, int wins, int losses, int draws) {
        this.uid = uid;
        this.displayName = displayName;
        this.email = email;
        this.wins = wins;
        this.losses = losses;
        this.draws = draws;
    }

    public String getUid() { return uid; }
    public void setUid(String uid) { this.uid = uid; }

    public String getDisplayName() { return displayName; }
    public void setDisplayName(String displayName) { this.displayName = displayName; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public int getWins() { return wins; }
    public void setWins(int wins) { this.wins = wins; }

    public int getLosses() { return losses; }
    public void setLosses(int losses) { this.losses = losses; }

    public int getDraws() { return draws; }
    public void setDraws(int draws) { this.draws = draws; }
}
```

GameResult.java (saved to `Results` collection)
```java
// package com.example.tictactoe.firebase;
public class GameResult {
    private String hostUid;       // host uid (document id in users)
    private String opponentName;  // opponent name (string only)
    private String result;        // "host", "opponent", or "draw"

    public GameResult() {}

    public GameResult(String hostUid, String opponentName, String result) {
        this.hostUid = hostUid;
        this.opponentName = opponentName;
        this.result = result;
    }

    public String getHostUid() { return hostUid; }
    public void setHostUid(String hostUid) { this.hostUid = hostUid; }

    public String getOpponentName() { return opponentName; }
    public void setOpponentName(String opponentName) { this.opponentName = opponentName; }

    public String getResult() { return result; }
    public void setResult(String result) { this.result = result; }
}
```

---

## 2) StartActivity — registration happens here (only this activity performs sign-up)

Purpose & theory
- StartActivity is the app entry point for registration. It runs when the app launches.
- If `FirebaseAuth.getInstance().getCurrentUser()` is not null, the user is already registered and StartActivity should immediately start NamesActivity (skip registration).
- If no current user, show a small registration form (email, password, displayName). On successful `createUserWithEmailAndPassword(...)` create the Firestore user document with:
  `FirebaseFirestore.getInstance().collection("users").document(uid).set(user)`
- After `.set(user)` succeeds, start NamesActivity.

Key code outline (simplified)
```java
FirebaseUser current = FirebaseAuth.getInstance().getCurrentUser();
if (current != null) {
    // user exists -> go to NamesActivity
    startActivity(new Intent(this, NamesActivity.class));
    finish();
} else {
    // show registration form
    // on register button:
    auth.createUserWithEmailAndPassword(email, password)
        .addOnSuccessListener(authResult -> {
            FirebaseUser firebaseUser = authResult.getUser();
            String uid = firebaseUser.getUid();
            String displayName = inputDisplayName.getText().toString().trim();
            User user = new User(uid, displayName, firebaseUser.getEmail(), 0, 0, 0);

            FirebaseFirestore.getInstance()
              .collection("users")
              .document(uid)
              .set(user) // <-- use .set(user) exactly as taught
              .addOnSuccessListener(aVoid -> {
                  // go to NamesActivity
                  startActivity(new Intent(this, NamesActivity.class));
                  finish();
              });
        });
}
```

Important student notes
- Registration runs once. On future app launches StartActivity sees the authenticated user and skips straight to NamesActivity.
- Use `.set(user)` exactly when creating `users/{uid}`.

---

## 3) NamesActivity — show host name and enter opponent name

Purpose & theory
- At this point the authenticated user exists (StartActivity ensured that).
- NamesActivity reads the host's user document from `users/{uid}` to show the host name in a non-editable TextView.
- The UI has: non-editable host name TextView, one EditText for opponent name, and a Start Game button.
- When Start Game is pressed pass hostUid, hostName and opponentName to GameActivity using an Intent.

How to read host name (example)
```java
FirebaseUser auth = FirebaseAuth.getInstance().getCurrentUser();
String hostUid = auth != null ? auth.getUid() : "";
FirebaseFirestore db = FirebaseFirestore.getInstance();

db.collection("users").document(hostUid)
  .get()
  .addOnSuccessListener(snapshot -> {
      if (snapshot.exists()) {
          User host = snapshot.toObject(User.class);
          tvHostName.setText(host != null ? host.getDisplayName() : "Host");
      } else {
          // doc missing (rare) — create a default or handle error
      }
  })
  .addOnFailureListener(e -> {
      Toast.makeText(this, "Error reading host data: " + e.getMessage(), Toast.LENGTH_LONG).show();
  });
```

Start the game (pass data via Intent)
```java
String hostName = tvHostName.getText().toString();
String opponentName = etOpponent.getText().toString().trim();

Intent i = new Intent(this, GameActivity.class);
i.putExtra("hostUid", hostUid);
i.putExtra("hostName", hostName);
i.putExtra("opponentName", opponentName);
startActivity(i);
```

Teacher decision you can make in class
- Decide whether host is always X or O — that is game logic only and does not affect Firestore.

---

## 4) GameActivity → FinishActivity (end of game) — compute result and pass it

At game end compute a `result` string:
- `"host"` if the host player won
- `"opponent"` if the second player won
- `"draw"` if the match is a draw

Pass the result and names to FinishActivity:
```java
Intent i = new Intent(this, FinishActivity.class);
i.putExtra("hostUid", hostUid);
i.putExtra("hostName", hostName);
i.putExtra("opponentName", opponentName);
i.putExtra("result", resultStr); // "host" | "opponent" | "draw"
startActivity(i);
finish();
```

---

## 5) FinishActivity — update `users/{uid}` counters and save a `Results` document

What you do (theory)
- Update the host's counters using `FieldValue.increment(1)` to add 1 to wins/losses/draws atomically.
- Save the game record to the `Results` collection. The `GameResult` contains `hostUid`, `opponentName` and `result`. Host name is not required in the `Results` doc (you already have the host's uid).

Example code
```java
String hostUid = getIntent().getStringExtra("hostUid");
String opponentName = getIntent().getStringExtra("opponentName");
String result = getIntent().getStringExtra("result"); // "host"|"opponent"|"draw"

FirebaseFirestore db = FirebaseFirestore.getInstance();

if (hostUid != null && !hostUid.isEmpty()) {
    DocumentReference hostRef = db.collection("users").document(hostUid);
    if ("host".equals(result)) {
        hostRef.update("wins", FieldValue.increment(1));
    } else if ("opponent".equals(result)) {
        hostRef.update("losses", FieldValue.increment(1));
    } else {
        hostRef.update("draws", FieldValue.increment(1));
    }
}

GameResult gr = new GameResult(hostUid, opponentName, result);
db.collection("Results")
  .add(gr)
  .addOnSuccessListener(docRef -> {
      Toast.makeText(this, "Result saved (id: " + docRef.getId() + ")", Toast.LENGTH_SHORT).show();
      // show final UI or return to NamesActivity
  })
  .addOnFailureListener(e -> {
      Toast.makeText(this, "Save failed: " + e.getMessage(), Toast.LENGTH_LONG).show();
  });
```

Theory note about FieldValue.increment
- `FieldValue.increment(1)` updates the numeric field on the server in an atomic way. If the field does not exist it is created with the increment value.

Atomicity option (advanced)
- If you want both the `Results` add and the user counter update to be atomic, use a `WriteBatch`:
  - create a new document reference with `collection("Results").document()` then `batch.set(resultRef, gr)` and `batch.update(hostRef, "wins", FieldValue.increment(1))`, then `batch.commit()`.
- For class simplicity, a separate `add(...)` then `update(...)` is fine.

---

## 6) How the host can view their game history (HistoryActivity idea)

Query `Results` where `hostUid == your uid`:
```java
String hostUid = FirebaseAuth.getInstance().getCurrentUser().getUid();
FirebaseFirestore.getInstance().collection("Results")
  .whereEqualTo("hostUid", hostUid)
  .get()
  .addOnSuccessListener(querySnapshot -> {
      List<GameResult> list = new ArrayList<>();
      for (DocumentSnapshot ds : querySnapshot.getDocuments()) {
          GameResult g = ds.toObject(GameResult.class);
          if (g != null) list.add(g);
      }
      // Show list in RecyclerView or ListView, e.g. "vs <opponentName> — Host won / Opponent won / Draw"
  });
```

---

## 7) Short security rules (ask your teacher to add these)
```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /Results/{docId} {
      allow create: if request.auth != null;
      allow read: if request.auth != null;
    }
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

---

Final student checklist
1. Implement `StartActivity` (one-time registration). On success write `users/{uid}` with `.set(user)` and go to `NamesActivity`.
2. Implement `NamesActivity` to read `users/{uid}` and show the host name, collect opponent name, and start the game passing hostUid/hostName/opponentName.
3. In `FinishActivity` update `users/{uid}` counters with `FieldValue.increment(...)` and add a `GameResult` to `Results`.
4. Implement a simple `HistoryActivity` that queries `Results` for `hostUid` to display past games.

If you want, I can now generate:
- `StartActivity.java` (full) — already provided earlier,
- `NamesActivity.java` (full),
- `FinishActivity.java` (full),
- `User.java` and `GameResult.java` files ready to drop into your project,
- A sample `HistoryActivity.java` to display results.

Tell me which files you want me to create next and I will add them. Good luck — keep the flow simple: register once, show host name, enter opponent name, play, save result.