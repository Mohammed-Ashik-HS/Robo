[03/12, 09:27] Mani 2: import cv2

import numpy as np 

import time
from DobotSDK import Dobot
# Initialize the Dobot object
dobot = Dobot(port='COM3') # Replace 'COM3' with your Dobot's actual COM port
# Connect to Dobot Magician 
if not dobot.connect():
print("Failed to connect to Dobot") 
exit()
# Define positions for each shape (circle, square, triangle) 
home_position = (200, 0, 50)
[03/12, 09:27] Mani 2: pick_position = (250, 50, 30)
shape_positions = { 
'circle': (150, -50, 30),
'square': (150, 50, 30),
'triangle': (100, -50, 30)
}
def identify_shape(contour):
"""Identify the shape based on the number of vertices in the contour.""" 
epsilon = 0.04 * cv2.arcLength(contour, True)
approx = cv2.approxPolyDP(contour, epsilon, True)
# Shape identification based on the number of vertices 
if len(approx) == 3:
return 'triangle'
elif len(approx) == 4:
# Check if it's a square by comparing aspect ratio 
x, y, w, h = cv2.boundingRect(approx) 
aspect_ratio = float(w) / h
if 0.95 < aspect_ratio < 1.05: # Aspect ratio close to 1 indicates a square 
return 'square'
else:
return 'circle' # Otherwise, classify it as a circle
# Capture video from the camera 
cap = cv2.VideoCapture(0)
if not cap.isOpened(): 
print("Cannot open camera") 
exit()
[03/12, 09:27] Mani 2: # Loop to continuously detect shapes and perform actions 
try:
while True:
ret, frame = cap.read() 
if not ret:
print("Failed to grab frame") 
break
# Convert the image to grayscale and apply thresholding 
gray = cv2.cvtColour(frame, cv2.COLOUR_BGR2GRAY) 
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blurred, 50, 150)
# Find contours
contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, 
cv2.CHAIN_APPROX_SIMPLE)
for contour in contours:
if cv2.contourArea(contour) > 1000: # Filter small contours
shape = identify_shape(contour)
# Draw the contour and shape name on the image 
cv2.drawContours(frame, [contour], -1, (0, 255, 0), 2) 
x, y, w, h = cv2.boundingRect(contour)
cv2.putText(frame, shape, (x, y - 10), cv2.FONT_HERSHEY_SIMPLEX, 1, (0,
255, 0), 2)
print(f"Detected shape: {shape}")
# Perform pick and place based on the identified shape
[03/12, 09:28] Mani 2: if shape:
# Move to Pick Position 
dobot.move_to(*pick_position, r=0) 
time.sleep(1)
# Activate gripper to pick the object 
dobot.set_gripper(True) 
time.sleep(1)
# Move to the corresponding shape position 
place_position = shape_positions[shape] 
dobot.move_to(*place_position, r=0) 
time.sleep(1)
# Release the object 
dobot.set_gripper(False) 
time.sleep(1)
# Return to home position 
dobot.move_to(*home_position, r=0) 
time.sleep(1)
# Display the frame for debugging 
cv2.imshow("Shape Detection", frame)
# Press 'q' to exit
if cv2.waitKey(1) & 0xFF == ord('q'): 
break
finally:
[03/12, 09:28] Mani 2: # Cleanup 
cap.release()
cv2.destroyAllWindows() 
dobot.disconnect()
print("Shape identification and pick-and-place operation completed.")
