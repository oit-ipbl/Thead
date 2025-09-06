# Thead
On this page you will learn how to use theading way. Students who have already learned opencv can start from this page.
## Theading
The following code is for using the Tello camera. Run the code to check Telle and OpenCV.
### sample01.py
```python
import cv2
from djitellopy import Tello

tello = Tello()
tello.connect()
tello.streamon()

while True:
    frame = tello.get_frame_read().frame
    frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
    cv2.imshow("Tello Video", frame)

    if cv2.waitKey(1) == ord('q'):
        break

tello.streamoff()
tello.end()
```
