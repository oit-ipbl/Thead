# Thread
To clear the final challenge, you need to use threads. Let's learn about threads in Tello through simple examples!
## Video and Move
Tello can smoothly move while sending videos by splitting threads.
### sample01.py
```python
from djitellopy import Tello
import cv2
import time
import threading

tello = Tello()
tello.connect()
print(f"Battery: {tello.get_battery()}%")
tello.streamon()
frame_read = tello.get_frame_read()

# camera function
def show_camera(end):
    while True:
        frame = frame_read.frame
        frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
        if frame is not None:
            cv2.imshow("Tello Camera", frame)
        if cv2.waitKey(1) & 0xFF == ord('q') :
            break
        if end == 0:
            break

# main
camera_thread = threading.Thread(target=show_camera,args=(1,), daemon=True)
camera_thread.start()
tello.takeoff()
tello.move_up(50)
tello.move_forward(100)
tello.rotate_clockwise(180)          
tello.move_forward(100)
tello.land()
tello.streamoff()
tello.end()
```
### sample01.py
