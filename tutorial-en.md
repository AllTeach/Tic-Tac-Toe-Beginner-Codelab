# Tic Tac Toe - Android Beginner Codelab

## Overview
Welcome to the Tic Tac Toe Android beginner codelab! In this tutorial, you will learn how to build a simple Tic Tac Toe game for Android using Java.

## Project Structure
This project follows the Model-View-Controller (MVC) architecture pattern:

### Model: TicTacToeModel.java
The Model represents the game logic and state. It contains:
- `board`: A 3x3 grid to store the game state
- `currentPlayer`: Tracks whose turn it is (X or O)

#### API Methods:
1. **Constructor**: Initializes the game board and sets the first player
2. **changePlayer()**: Switches between X and O players
3. **isLegal(int row, int col)**: Checks if a move is valid
4. **makeMove(int row, int col)**: Places the current player's mark on the board
5. **checkWin()**: Checks if the current player has won (to be implemented)
6. **isTie()**: Checks if the game is a tie (to be implemented)
7. **resetGame()**: Resets the board for a new game
8. **getCurrentPlayer()**: Returns the current player (X or O)

### View: activity_main.xml
The View defines the user interface using XML. It includes:
- A GridLayout containing 9 buttons arranged in a 3x3 grid
- Each button has a `tag` attribute with its row and column position (e.g., "0,0")
- Each button has an `onClick` attribute that calls the `onCellClick` method

### Controller: MainActivity.java
The Controller (Activity) connects the Model and View. It:
- Creates an instance of TicTacToeModel
- Handles button clicks through the `onCellClick` method
- Updates the UI based on the game state
- Shows Toast messages for wins and ties

## How to Play
1. The game starts with player X
2. Players take turns clicking on empty cells
3. The first player to get 3 marks in a row (horizontally, vertically, or diagonally) wins
4. If all cells are filled and no player has won, the game is a tie
5. The game resets automatically after a win or tie

## Your Task
The skeleton code is provided with TODO comments for the following methods in TicTacToeModel.java:
- `checkWin()`: Implement the logic to check if the current player has won
- `isTie()`: Implement the logic to check if the game is a tie

### Hints for checkWin():
- Check all 3 rows
- Check all 3 columns
- Check both diagonals
- Return true if any line has three matching marks

### Hints for isTie():
- Check if all cells are filled
- Make sure there's no winner
- Return true only if the board is full and nobody won

## Running the Project
1. Open the project in Android Studio
2. Connect an Android device or start an emulator
3. Click the Run button
4. Test the game by clicking on cells

Good luck and have fun coding!
