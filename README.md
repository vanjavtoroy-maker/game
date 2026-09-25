import tkinter as tk
import random

root = tk.Tk()
root.title("Поймай Урожай: Нарастающий вызов!")

WIDTH = 600   
HEIGHT = 600  

canvas = tk.Canvas(root, width=WIDTH, height=HEIGHT)
canvas.pack()

BASE_SPEED = 4       
CURRENT_DELAY = 45   

bg_image = tk.PhotoImage(file="img/tree.png")
bg_id = canvas.create_image(0, 0, image=bg_image, anchor="nw")

score = 0
lives = 3
game_active = True

score_text = canvas.create_text(60, 20, text="Счёт: 0", font=("Arial", 16, "bold"), fill="white")
difficulty_text = canvas.create_text(WIDTH // 2, 20, text="Сложность: 1", font=("Arial", 14, "italic"), fill="yellow")
lives_text = canvas.create_text(WIDTH - 70, 20, text="❤️❤️❤️", font=("Arial", 16), fill="red")

basket_x = WIDTH // 2       
basket_y = HEIGHT - 50      

basket = canvas.create_text(basket_x, basket_y, text="🧺", font=("Arial", 40), fill="brown")
objects = []

def create_object():
    if not game_active:
        return

    x = random.randint(40, WIDTH - 40)
    y = 30
    rand_val = random.randint(1, 10)
    
    if rand_val <= 5:        
        item = canvas.create_text(x, y, text="🍎", font=("Arial", 25))
        object_type = "apple"
    elif rand_val <= 7:      
        item = canvas.create_text(x, y, text="🍐", font=("Arial", 25))
        object_type = "pear"
    elif rand_val <= 9:      
        item = canvas.create_text(x, y, text="🥫", font=("Arial", 25))
        object_type = "trash"
    else:                    
        item = canvas.create_text(x, y, text="💣", font=("Arial", 25))
        object_type = "bomb"

    objects.append([item, x, y, object_type])
    spawn_delay = max(400, 1200 - (score * 30))
    root.after(random.randint(spawn_delay - 200, spawn_delay + 200), create_object)

def move_left(event):
    global basket_x
    if game_active and basket_x > 40:
        basket_x -= 25
        canvas.move(basket, -25, 0)

def move_right(event):
    global basket_x
    if game_active and basket_x < WIDTH - 40:
        basket_x += 25
        canvas.move(basket, 25, 0)

def lose_life(count=1):
    global lives, game_active
    lives -= count
    if lives < 0:
        lives = 0
    canvas.itemconfig(lives_text, text="❤️" * lives)
    if lives <= 0:
        game_active = False
        end_game()

def update_game():
    global score
    if not game_active:
        return

    current_speed = BASE_SPEED + (score // 5)
    current_delay = max(20, CURRENT_DELAY - (score // 3) * 2)
    level = 1 + (score // 5)
    canvas.itemconfig(difficulty_text, text=f"Сложность: {level}")

    for obj in objects[:]:
        item = obj[0]
        x = obj[1]
        y = obj[2]
        object_type = obj[3]

        y += current_speed
        obj[2] = y
        canvas.move(item, 0, current_speed)

        if basket_y - 25 <= y <= basket_y + 25:
            if abs(x - basket_x) < 45:
                if object_type == "apple":
                    score += 1
                elif object_type == "pear":
                    score += 2
                elif object_type == "trash":
                    lose_life(1)
                elif object_type == "bomb":
                    score = max(0, score - 2)
                    lose_life(1)

                canvas.itemconfig(score_text, text=f"Счёт: {score}")
                canvas.delete(item)
                objects.remove(obj)
                continue

        if y > HEIGHT:
            if object_type == "apple" or object_type == "pear":
                lose_life(1)
            canvas.delete(item)
            objects.remove(obj)

    if game_active:
        root.after(current_delay, update_game)

def end_game():
    canvas.delete("all")
    canvas.create_rectangle(0, 0, WIDTH, HEIGHT, fill="#2c3e50")
    canvas.create_text(WIDTH // 2, HEIGHT // 2 - 40, text="ИГРА ОКОНЧЕНА! ❌", font=("Arial", 28, "bold"), fill="#e74c3c")
    canvas.create_text(WIDTH // 2, HEIGHT // 2 + 30, text=f"Вы потеряли все жизни.\nВаш итоговый счёт: {score}\nВы добрались до уровня сложности: {1 + (score // 5)}", font=("Arial", 18), fill="white", justify="center")

root.bind("<Left>", move_left)
root.bind("<Right>", move_right)

create_object()
update_game()
root.mainloop()
