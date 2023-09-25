# Hand-Filter
Python script uses the OpenCV and MediaPipe libraries to create a real-time hand gesture recognition and background replacement application.
Initialization:

The code starts by importing the necessary libraries, including OpenCV and MediaPipe, which is used for hand tracking.
MediaPipe's hand tracking module is initialized, along with other variables like the video capture device and background images.
Frame Capture:

A while loop is set up to continuously capture frames from the video feed. The video feed is configured to use the user's default camera with specific width and height settings.
Hand Tracking:

Within the loop, each captured frame is flipped horizontally using cv2.flip to create a mirror effect, which makes it more intuitive for users.
The frame is then converted from the BGR color format to RGB using cv2.cvtColor.
The hands.process function from MediaPipe is used to process the RGB image and detect hand landmarks in the frame.
Hand Landmark Processing:

If hand landmarks are detected in the frame, the code iterates through each detected hand.
For each hand, it extracts the landmarks' pixel coordinates and calculates various hand-related measurements:
position_data function extracts the coordinates of key points on the hand, including wrist, thumb tip, and finger joints.
The distance between the wrist and the index finger's metacarpophalangeal (MCP) joint is used to estimate the size of the palm.
The distance between the index finger's tip and the pinky finger's tip is calculated.
A ratio is computed by dividing the above two distances. This ratio is used to determine if the hand is open or closed.
Magic Circle Overlay:

If the hand's ratio suggests an open hand, the code proceeds to overlay magic circle images on the video frame.
It calculates the position and size of the shield (magic circle) based on the hand's palm size and position.
The shield is rotated by a certain degree (deg) to create an animated effect.
Transparent images of magic circles (img_1 and img_2) are overlaid onto the frame at the calculated position and with the calculated size using the transparent function.
Background Switching:

The code also checks if both hands are open. If they are, it switches between different background images by changing the background_flag.
A message indicating whether hands are open or closed is displayed on the frame using cv2.putText.
Display:

The processed frame, which now includes the overlay of magic circles and the selected background, is displayed using cv2.imshow.
User Interaction:

The program runs until the user presses the 'q' key, at which point the loop is exited.
Cleanup:

Finally, the video capture is released using video.release(), and all OpenCV windows are closed with cv2.destroyAllWindows().
