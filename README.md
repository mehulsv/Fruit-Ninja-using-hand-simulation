import os
import cv2
import mediapipe as mp
import pygame
import random
import time

# Initialize Mediapipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1)
mp_draw = mp.solutions.drawing_utils

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 640, 480
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Fruit Ninja with Hand Tracking")

# Define the base path for images
base_path = 'c:\\Users\\Sajeev\\Desktop\\CGProject\\CGProject\\'

# Load images
def load_image(filename, size=None):
    path = os.path.join(base_path, filename)
    if os.path.exists(path):
        image = pygame.image.load(path)
        if size:
            image = pygame.transform.scale(image, size)
        return image    
    else:
        print(f"Image not found: {path}")
        return None

# Dictionary to hold fruit images
fruit_imgs = {
    'apple': load_image('apple.png', (75, 75)),
    'banana': load_image('banana.png', (75, 75)),
    'orange': load_image('orange.png', (75, 75))
}
bg_img = load_image('game bg.png', (WIDTH, HEIGHT))

# Remove any None images
fruit_imgs = {k: v for k, v in fruit_imgs.items() if v is not None}

# Define points for each fruit
fruit_points = {
    'apple': 2,
    'banana': 3,
    'orange': 5
}

# Variables
running = True
fruits = []
score = 0
lives = 5
font = pygame.font.Font(None, 36)
state = "START"  # Possible states: START, PLAY, PAUSE, GAME_OVER
sliced_fruits = []

# Function to spawn a fruit
def spawn_fruit():
    x = random.randint(0, WIDTH - 50)
    y = HEIGHT
    name, img = random.choice(list(fruit_imgs.items()))
    fruits.append([x, y, name, img])

# Function to reset the game
def reset_game():
    global fruits, score, lives, sliced_fruits
    fruits = []
    score = 0
    lives = 5
    sliced_fruits = []

# Main loop
cap = cv2.VideoCapture(0)
if not cap.isOpened():
    print("Error: Could not open webcam.")
    running = False

last_spawn_time = time.time()

while running:
    success, img = cap.read()
    if not success:
        print("Error: Could not read frame from webcam.")
        break

    img = cv2.flip(img, 1)
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    # Detect hands
    results = hands.process(img_rgb)
    hand_positions = []
    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)
            for id, lm in enumerate(hand_landmarks.landmark):
                h, w, c = img.shape
                cx, cy = int(lm.x * w), int(lm.y * h)
                if id == 4:
                    hand_positions.append((cx, cy))

    # Convert img to Pygame surface
    frame = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    frame = cv2.transpose(frame)
    frame = pygame.surfarray.make_surface(frame)
    screen.blit(frame, (0, 0))

    if state == "START":
        screen.blit(bg_img, (0, 0))
        start_text = font.render("Press SPACE to Start", True, (255, 255, 255))
        screen.blit(start_text, (WIDTH // 2 - start_text.get_width() // 2, HEIGHT // 2 - start_text.get_height() // 2))

    elif state == "PLAY":
        if time.time() - last_spawn_time > 1:
            spawn_fruit()
            last_spawn_time = time.time()

        for fruit in fruits[:]:
            fruit[1] -= 5
            screen.blit(fruit[3], (fruit[0], fruit[1]))
            if fruit[1] < 0:
                fruits.remove(fruit)
                lives -= 1
                if lives == 0:
                    state = "GAME_OVER"

        # Check for slicing
        for pos in hand_positions:
            for fruit in fruits[:]:
                fx, fy, name, img = fruit
                if fx < pos[0] < fx + 50 and fy < pos[1] < fy + 50:
                    fruits.remove(fruit)
                    points = fruit_points[name]
                    score += points
                    sliced_fruits.append([fx, fy, points, time.time()])

        # Draw score and lives
        score_text = font.render(f"Score: {score}", True, (0, 0, 139))
        lives_text = font.render(f"Lives: {lives}", True, (0, 0, 139))
        screen.blit(score_text, (10, 10))
        screen.blit(lives_text, (10, 50))

        # Draw points for each sliced fruit
        current_time = time.time()
        for sf in sliced_fruits[:]:
            if current_time - sf[3] > 1:
                sliced_fruits.remove(sf)
            else:
                elapsed_time = current_time - sf[3]
                new_y = sf[1] - 50 + int(elapsed_time * -50)
                color_value = max(255 - int(elapsed_time * 255), 0)
                points_text = font.render(f"+{sf[2]}", True, (color_value, color_value, 0))
                screen.blit(points_text, (sf[0], new_y))

    elif state == "PAUSE":
        screen.blit(bg_img, (0, 0))
        pause_text = font.render("Paused. Press SPACE to Resume", True, (255, 255, 255))
        screen.blit(pause_text, (WIDTH // 2 - pause_text.get_width() // 2, HEIGHT // 2 - pause_text.get_height() // 2))

    elif state == "GAME_OVER":
        screen.blit(bg_img, (0, 0))
        game_over_text = font.render("Game Over! Press SPACE to Restart", True, (255, 255, 255))
        screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 2 - game_over_text.get_height() // 2))
        final_score_text = font.render(f"Final Score: {score}", True, (255, 255, 255))
        screen.blit(final_score_text, (WIDTH // 2 - final_score_text.get_width() // 2, HEIGHT // 2 + game_over_text.get_height() // 2))

    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if state == "START":
                    state = "PLAY"
                elif state == "PLAY":
                    state = "PAUSE"
                elif state == "PAUSE":
                    state = "PLAY"
                elif state == "GAME_OVER":
                    reset_game()
                    state = "START"

    pygame.display.update()

cap.release()
pygame.quit()
cv2.destroyAllWindows()
