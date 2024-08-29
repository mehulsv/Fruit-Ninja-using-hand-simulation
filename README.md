Fruit Ninja with Hand Tracking
This project is a simple implementation of the popular game "Fruit Ninja," enhanced with hand tracking using Mediapipe and Pygame. The game captures hand movements from a webcam and uses them to slice fruits on the screen.

Features
Hand Tracking: Utilizes Mediapipe to detect hand movements and identify the index finger position for slicing fruits.
Multiple Fruits: Spawns different types of fruits with varying point values.
Scoring System: Players earn points by slicing fruits and lose lives if fruits fall off the screen without being sliced.
Game States: Includes multiple game states: START, PLAY, PAUSE, and GAME OVER.
Requirements
Python 3.x
OpenCV
Mediapipe
Pygame
Installation
Install Python: Ensure Python 3.x is installed on your system.
Install Dependencies:
sh
Copy code
pip install opencv-python mediapipe pygame
Download or Clone the Repository:
sh
Copy code
git clone <repository_url>
Set Image Path: Ensure the images (fruits and background) are in the directory c:\Users\Sajeev\Desktop\CGProject\CGProject\.
How to Run
Navigate to the project directory:
sh
Copy code
cd c:\Users\Sajeev\Desktop\CGProject\CGProject\
Run the game:
sh
Copy code
python fruit_ninja.py
Controls:
Press SPACE to start the game.
Use your hand to slice the fruits on the screen.
Press SPACE to pause and resume the game.
Press ESC or close the window to exit.
Game Logic
Fruit Spawning: Fruits spawn randomly from the bottom of the screen and move upwards.
Slicing Fruits: Detects the player's index finger to slice fruits. Points are awarded based on the type of fruit.
Game Over: The game ends when the player loses all lives.
Known Issues
The game may not run smoothly on all systems, depending on the webcam quality and system specifications.
Future Improvements
Add more fruit types and special items.
Implement more advanced hand gestures.
Improve graphics and animations.
