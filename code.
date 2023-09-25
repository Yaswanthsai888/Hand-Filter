import cv2
import mediapipe as mp

# Initialize MediaPipe hands module
mpHands = mp.solutions.hands
hands = mpHands.Hands()
mpDraw = mp.solutions.drawing_utils

# Initialize the video capture
video = cv2.VideoCapture(0)
video.set(3, 1280)
video.set(4, 960)

# Load background images
background_images = [
    cv2.imread('images/abcd.png'),
    cv2.imread('images/magic_circle_2.png')
]

# Load magic circle images with transparency
img_1 = cv2.imread('images/magic_circle_1.png', -1)
img_2 = cv2.imread('images/magic_circle_ccw.png', -1)

# Initialize background flag and rotation degree
background_flag = 0
deg = 0.3

# Function to extract hand landmark positions
def position_data(imlist):
    global wrist, thumb_tip, index_mcp, index_tip, midle_mcp, midle_tip, ring_tip, pinky_tip
    wrist = (imlist[0][0], imlist[0][1])
    thumb_tip = (imlist[4][0], imlist[4][1])
    index_mcp = (imlist[5][0], imlist[5][1])
    index_tip = (imlist[8][0], imlist[8][1])
    midle_mcp = (imlist[9][0], imlist[9][1])
    midle_tip = (imlist[12][0], imlist[12][1])
    ring_tip = (imlist[16][0], imlist[16][1])
    pinky_tip = (imlist[20][0], imlist[20][1])

# Function to draw a line between two points
def draw_line(p1, p2, size=5):
    cv2.line(img, p1, p2, (50, 50, 255), size)
    cv2.line(img, p1, p2, (225, 225, 255), round(size/2))

# Function to calculate the distance between two points
def calculated_distance(p1, p2):
    x1, y1, x2, y2 = p1[0], p1[1], p2[0], p2[1]
    length = ((x2-x1)**2 + (y2-y1)**2)**(1/2)
    return length

# Function to overlay a transparent image on another image
def transparent(targetimg, x, y, size=None):
    if size is not None:
        targetimg = cv2.resize(targetimg, size)

    newFrame = img.copy()
    b, g, r, a = cv2.split(targetimg)
    overlay_color = cv2.merge((b, g, r))
    mask = cv2.medianBlur(a, 1)
    h, w, _ = overlay_color.shape
    roi = newFrame[y:y+h, x:x+w]

    img1_bg = cv2.bitwise_and(roi.copy(), roi.copy(), mask=cv2.bitwise_not(mask))
    img2_fg = cv2.bitwise_and(overlay_color, overlay_color, mask=mask)
    newFrame[y:y+h, x:x+w] = cv2.add(img1_bg, img2_fg)

    return newFrame

while True:
    ret, img = video.read()
    img = cv2.flip(img, 1)
    rgbimg = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    result = hands.process(rgbimg)

    # Initialize variables to track if both hands are open
    left_hand_open = False
    right_hand_open = False

    if result.multi_hand_landmarks:
        for hand in result.multi_hand_landmarks:
            imList = []
            for id, im in enumerate(hand.landmark):
                h, w, c = img.shape
                coorx, coory = int(im.x*w), int(im.y*h)
                imList.append([coorx, coory])

            position_data(imList)
            palm = calculated_distance(wrist, index_mcp)
            distance = calculated_distance(index_tip, pinky_tip)
            ratio = distance / palm

            if ratio > 1.0:
                centerx = midle_mcp[0]
                centery = midle_mcp[1]
                shield_size = 2.0
                diameter = round(palm*shield_size)
                x1 = round(centerx - (diameter/2))
                y1 = round(centery - (diameter/2))
                h, w, c = img.shape
                if x1 < 0:
                    x1 = 0
                elif x1 > w:
                    x1 = w
                if y1 < 0:
                    y1 = 0
                elif y1 > h:
                    y1 = h
                if x1 + diameter > w:
                    diameter = w - x1
                if y1 + diameter > h:
                    diameter = h - y1
                shield_size = diameter, diameter
                ang_vel = 1.2
                deg = deg + ang_vel
                if deg > 360:
                    deg = 0
                hei, wid, col = img_1.shape
                cen = (wid//2, hei//2)
                m1 = cv2.getRotationMatrix2D(cen, round(deg), 1.0)
                m2 = cv2.getRotationMatrix2D(cen, round(360-deg), 1.0)
                rotated1 = cv2.warpAffine(img_1, m1, (wid, hei))
                rotated2 = cv2.warpAffine(img_2, m2, (wid, hei))
                if diameter != 0:
                    img = transparent(rotated1, x1, y1, shield_size)
                    img = transparent(rotated2, x1, y1, shield_size)

            # Check if both hands are open
            if ratio > 1.0:
                if hand == result.multi_hand_landmarks[0]:
                    left_hand_open = True
                else:
                    right_hand_open = True

    # Check if both hands are open to switch the background
    if left_hand_open and right_hand_open:
        background_flag = (background_flag + 1) % len(background_images)
        cv2.putText(img, "Hands Open", (10, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
    else:
        cv2.putText(img, "Hands Closed", (10, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
        # Set the background to be transparent if no hands are detected
        background = cv2.imread('images/transparent.png')  # Load a transparent image or create one
        if background is not None:
            background = cv2.resize(background, (img.shape[1], img.shape[0]))  # Resize to match img dimensions
            if background.shape[:2] == img.shape[:2]:  # Check if dimensions match
                img = cv2.addWeighted(img, 1.3, background, 0.3, 0)

    # Apply the selected background to the frame
    img_height, img_width, _ = img.shape
    background = cv2.resize(background_images[background_flag], (img_width, img_height))
    img = cv2.addWeighted(img, 1.3, background, 0.3, 0)

    cv2.imshow("Image", img)
    k = cv2.waitKey(1)
    if k == ord('q'):
        break

# Release video capture and close all windows
video.release()
cv2.destroyAllWindows()
