Smart Retail project 
============

**In this project, we developed a smart refrigerator**



Project detail is explained in the following section


Table of contents
==================

<!--ts-->
* [Integration](#Integration)
    * [Raspberri_pi](#Raspberri_pi)
    * [Camera](#Camera)
    * [Load_Cell](#Load_Cell)
* [Test](#Test)
* [Others](#Others)
    * [DB & AWS](#DB & AWS )
    * [Door close & open api]()
   
<!--te-->


Integration
===========

In this project, we integrated load cell, cameras, raspberry pi to make a smart refrigerator like Amazon's smart convenience store.

Raspberri_pi
=============

First step is to setup raspberry pi to each floors and conect to internet so that we can use ip to connet with `master server` and `redis server`.


Camera
=======

After completing reaspberry pi setup, we adjust cameras in each floor and connect to raspberry pi. Based on our requirement, we setup cameras in different angles and positions.

Here,`Master respberry pi` handles all captures and sends to redis to test image `classification` model. [`Camera python file`]() setup :

```
def cam_set(cap):
    width = 3840
    height = 2160
    
    cap_AUTOFOCUS = 0
    cap_FOCUS = 0
    
    #cap.set(cv2.CAP_PROP_FOURCC, MJPG_CODEC)
    cap.set(cv2.CAP_PROP_AUTOFOCUS, cap_AUTOFOCUS)
    cap.set(cv2.CAP_PROP_FOCUS, cap_FOCUS)
    
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, width)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, height)
    cap.read()

def cam_list():
    # check cam plugged in
    camera_list = []
    get_all_device_list = Popen(["v4l2-ctl --list-devices"], shell=True, stdout=PIPE, encoding="utf-8")
    stdout, stderr = get_all_device_list.communicate()
    get_all_device_list = stdout.split('\n\n')
    for i in get_all_device_list:
        if "bcm2835-codec-decode" in i:
            continue
        if i != '':
            item = i.split('\n\t')
            camera_list.append(item[1])
    print(camera_list)
    return camera_list
    
   ```

Load_Cell
==========

Since each floor of each refrigerator has 8 columns with different products, we used load cell to optimize the weight of each columns separately. Sample images of floors are shown below. 

**Images**

We setup raspberri pi in each floor. Each raspberry pi has a python file which gives an information of weight and sends to redis server to analyze the output with deep learning model's result. 

Connecting with server and getting ip: 

```
#-*- coding: utf-8 -*-
import re
import json
import redis
import config
import socket
import serial
from keys import keys
from multiprocessing import Process

r = redis.Redis(host='  ip ', port=6379, db=2,
                username='worker', password=keys.get('redis', './keys'))

PORT = '/dev/ttyUSB0'
BAUDRATE = 38400
ser = serial.Serial(PORT, BAUDRATE, timeout=1)

# get ip
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.connect(("8.8.8.8", 80))
get_eth0_ip = s.getsockname()[0]
ip_end = get_eth0_ip.split('.')[-1]
s.close()
```

Similarly we define the device id:

```
store_id = config.refrigerators['storeId']
for d in config.refrigerators['device_list']:
    if ip_end in d['floors'].keys():
        device_id = d['deviceId']
        floor = d['floors'][ip_end]
```

After completing setup, we do [calibration]()

<table border="0">
   <tr>
      <td>
      <img src="./Source/Data_collection/08181_product_w_1_1.png" width="100%" />
      </td>
      <td>
      <img src="./Source/Data_collection/08181_product_w_2_1.png" width="100%" />
      </td>
      <td>
      <img src="./Source//Data_collection/08181_product_w_2_3.png" width="100%" />
      </td>
   </tr>
   </table>




Test
=====
 Main part of the project is to combine two results (load cell and AI) and visualize it automatically on the screen when customer buys something.
**Images**


**Results Integrations**

We used `image classification` model [`efficientnet`]() to classify products of each columns. We only classify the front line of each floor. 
After classifing each products from `redis server` we combine this result with load cell `weight result`. `Load cell` result analyzes weight of each column based on each classifying result. Whenever there is minus and plus in the columns, it shows the removing and adding produt in the result. 

**Images**




**project is going on **






 
 
 
