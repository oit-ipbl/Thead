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
## Find red
Let's confirm that using threads allows Tello to move smoothly.
### sample02.py
```python
from djitellopy import Tello
import cv2
import time
import threading

tello = Tello()
tello.connect()
print(f"Battery: {tello.get_battery()}%")
tello.streamon()

stop_event = threading.Event()
state_event =threading.Event()
def show_camera(end):
    while True:
        frame = tello.get_frame_read().frame
        frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
        frame = cv2.resize(frame, (720, 480))
        
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        
        hsv_min = (0, 70, 70)
        hsv_max = (15, 255, 255)
        mask1 = cv2.inRange(hsv, hsv_min, hsv_max)
    
        hsv_min = (160, 70, 70)
        hsv_max = (179, 255, 255)
        mask2 = cv2.inRange(hsv, hsv_min, hsv_max)
    
        mask = mask1 + mask2
        
        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        

        if contours:

            largest_contour = max(contours, key=cv2.contourArea)
            if cv2.contourArea(largest_contour) > 1000: 
                x, y, w, h = cv2.boundingRect(largest_contour)
                cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 0, 255), 2)
                print(cv2.contourArea(largest_contour) )
                print(x+w/2,y+h/2)
                print("fond color")
                state_event.set() 
            else:
                print("nothing")
                state_event.clear()
        cv2.imshow("Tracking", frame)
        key = cv2.waitKey(1)
        if key == ord('q'):
            stop_event.set() 

camera_thread = threading.Thread(target=show_camera,args=(1,), daemon=True)
camera_thread.start()

tello.takeoff()
while not stop_event.is_set():
    if state_event.is_set():
        tello.send_rc_control(0, 0, 0, 0)
    else:
        tello.rotate_clockwise(30)
    time.sleep(1.0)
tello.land()
tello.streamoff()
tello.end()
```
