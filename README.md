import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk

# To track cells that are already part of a scored line
scored_cells = set()

# Define the colors for each line built by a player
line_colors = {"Uzumaki Clan": ["green", "blue", "red"], "Uchiha Clan": ["green", "blue", "red"]}

# Track the number of lines each player has made
player_line_count = {"Uzumaki Clan": 0, "Uchiha Clan": 0}

# Function to check for a winner and update scores
def check_winner():
    global player_scores

    # Check for horizontal three-symbol lines
    for i in range(board_size):
        for j in range(board_size - 2):  # Subtract 2 to prevent index out of range
            if (board[i][j] == board[i][j+1] == board[i][j+2] == current_player and
                (i, j) not in scored_cells and 
                (i, j+1) not in scored_cells and 
                (i, j+2) not in scored_cells):
                color = line_colors[current_player][min(player_line_count[current_player], 2)]
                highlight_winning_line([(i, j), (i, j+1), (i, j+2)], color)
                player_scores[current_player] += 1
                scored_cells.update([(i, j), (i, j+1), (i, j+2)])
                player_line_count[current_player] += 1
    
    # Check for vertical three-symbol lines
    for i in range(board_size - 2):  # Subtract 2 to prevent index out of range
        for j in range(board_size):
            if (board[i][j] == board[i+1][j] == board[i+2][j] == current_player and
                (i, j) not in scored_cells and 
                (i+1, j) not in scored_cells and 
                (i+2, j) not in scored_cells):
                color = line_colors[current_player][min(player_line_count[current_player], 2)]
                highlight_winning_line([(i, j), (i+1, j), (i+2, j)], color)
                player_scores[current_player] += 1
                scored_cells.update([(i, j), (i+1, j), (i+2, j)])
                player_line_count[current_player] += 1

    # Check for diagonal three-symbol lines
    for i in range(board_size - 2):  # Subtract 2 to prevent index out of range
        for j in range(board_size - 2):
            if (board[i][j] == board[i+1][j+1] == board[i+2][j+2] == current_player and
                (i, j) not in scored_cells and 
                (i+1, j+1) not in scored_cells and 
                (i+2, j+2) not in scored_cells):
                color = line_colors[current_player][min(player_line_count[current_player], 2)]
                highlight_winning_line([(i, j), (i+1, j+1), (i+2, j+2)], color)
                player_scores[current_player] += 1
                scored_cells.update([(i, j), (i+1, j+1), (i+2, j+2)])
                player_line_count[current_player] += 1
            if (board[i+2][j] == board[i+1][j+1] == board[i][j+2] == current_player and
                (i+2, j) not in scored_cells and 
                (i+1, j+1) not in scored_cells and 
                (i, j+2) not in scored_cells):
                color = line_colors[current_player][min(player_line_count[current_player], 2)]
                highlight_winning_line([(i+2, j), (i+1, j+1), (i, j+2)], color)
                player_scores[current_player] += 1
                scored_cells.update([(i+2, j), (i+1, j+1), (i, j+2)])
                player_line_count[current_player] += 1

    update_score_display()

# Function to highlight the winning line with the appropriate color
def highlight_winning_line(combo, color):
    for position in combo:
        row, col = position
        buttons[row][col].config(bg=color)

# Function to handle button clicks during the game
def button_click(row, col):
    global current_player, winner
    if board[row][col] == "" and winner is None:
        board[row][col] = current_player
        buttons[row][col].config(image=x_img if current_player == "Uzumaki Clan" else o_img)
        
        check_winner()

        if player_scores[current_player] >= 3:
            winner = current_player
            messagebox.showinfo("Game Over", f"{winner} wins with {player_scores[winner]} points!")
            reset_game()
        elif all(board[row][col] != "" for row in range(board_size) for col in range(board_size)):
            messagebox.showinfo("Tic-Tac-Toe", "It's a draw!")
            reset_game()
        else:
            current_player = "Uchiha Clan" if current_player == "Uzumaki Clan" else "Uzumaki Clan"
            update_next_player_display()

# Function to update the score display
def update_score_display():
    score_label.config(text=f"Uzumaki Clan: {player_scores['Uzumaki Clan']} | Uchiha Clan: {player_scores['Uchiha Clan']}")

# Function to update which player's turn is next
def update_next_player_display():
    next_player_label.config(text=f"Next Player: {current_player}")

# Function to reset the game board
def reset_game():
    global board, current_player, winner, scored_cells, player_line_count
    board = [["" for _ in range(board_size)] for _ in range(board_size)]
    scored_cells = set()
    player_line_count = {"Uzumaki Clan": 0, "Uchiha Clan": 0}
    for i in range(board_size):
        for j in range(board_size):
            buttons[i][j].config(image='', bg="SystemButtonFace")
    current_player = "Uzumaki Clan"
    winner = None
    player_scores["Uzumaki Clan"] = 0
    player_scores["Uchiha Clan"] = 0
    update_score_display()
    update_next_player_display()

# Function to create the game board
def create_board():
    global buttons, board, current_player, winner, player_scores, next_player_label
    buttons = [[None]*board_size for _ in range(board_size)]
    board = [[""]*board_size for _ in range(board_size)]
    current_player = "Uzumaki Clan"
    winner = None
    player_scores = {"Uzumaki Clan": 0, "Uchiha Clan": 0}
    scored_cells = set()  # Reset the scored cells for the new game
    player_line_count = {"Uzumaki Clan": 0, "Uchiha Clan": 0}  # Reset the line counts for the new game
    for row in range(board_size):
        for col in range(board_size):
            buttons[row][col] = tk.Button(root, image='', font=("Arial", 24), width=4, height=2,
                                          command=lambda r=row, c=col: button_click(r, c))
            buttons[row][col].place(x=col*100+50, y=row*100+50, width=100, height=100)
    update_score_display()
    update_next_player_display()

# Initialize game variables
board_size = 7  # Set the board size to 7x7 for more space

# Setup the GUI
root = tk.Tk()
root.title("Tic-Tac-Toe")
root.geometry(f"{board_size*100+100}x{board_size*100+200}")

# Load the background image
bg_image = Image.open(r"C:\Users\GLER\Desktop\project\background.png")
bg_image = bg_image.resize((board_size*100+100, board_size*100+200), Image.Resampling.LANCZOS)
bg_photo = ImageTk.PhotoImage(bg_image)

# Create a Canvas to hold the background image
canvas = tk.Canvas(root, width=board_size*100+100, height=board_size*100+200)
canvas.create_image(0, 0, anchor=tk.NW, image=bg_photo)
canvas.place(x=0, y=0)

# Load the Uzumaki and Uchiha images
x_img = ImageTk.PhotoImage(Image.open(r"C:\Users\GLER\Desktop\project\uzumaki.png").resize((100, 100), Image.Resampling.LANCZOS))
o_img = ImageTk.PhotoImage(Image.open(r"C:\Users\GLER\Desktop\project\uchiha.png").resize((100, 100), Image.Resampling.LANCZOS))

# Frame to hold the bottom UI elements
bottom_frame = tk.Frame(root, bg="#f0f0f0")  # Add a slight background color to make it stand out
bottom_frame.place(x=50, y=board_size*100+100, width=board_size*100)

# Display the score for both clans
score_label = tk.Label(bottom_frame, text="", font=("Arial", 16), bg="#f0f0f0")
score_label.pack(side=tk.LEFT, padx=10)

# Display which player's turn is next
next_player_label = tk.Label(bottom_frame, text="", font=("Arial", 16), bg="#f0f0f0")
next_player_label.pack(side=tk.LEFT, padx=10)

# Add the reset button to reset the game
reset_button = tk.Button(bottom_frame, text="Reset", font=("Arial", 15), command=reset_game)
reset_button.pack(side=tk.RIGHT, padx=20)

# Create the initial game board
create_board()

# Start the main event loop
root.mainloop()

