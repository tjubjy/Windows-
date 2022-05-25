# Windows-OpenCV-ffmpeg-livego-
采用OpenCV+ffmpeg+livego搭建,python编写,ffmpeg实现推流,livego用作流媒体转发的简单直播系统.  可以用OpenCV打开本地媒体文件,推流给服务器,也可以打开本地摄像头或者是IP摄像头.



代码如下,有比较详细的注释

import cv2
import time
import subprocess

#推流的地址
# 说明: 启动livego,用浏览器访问http://ip:8090/control/get?room=movie 得到channelkey
# rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk 就是channelkey
# rtmp地址格式 : rtmp://ip:1935/{appname}/{channelkey}
# 接收端访问rtmp视频流地址:
rtmp = 'rtmp://192.168.3.3:1935/live/rfBd56ti2SMtYvSgD5xAV0YU99zampta7Z7S575KLkIZ9PYk'

#IPCameraUrl = 'rtsp://admin:admin@192.168.3.20:8554/live'
#cap = cv2.VideoCapture(path) #path可以填本地的一个媒体文件路径 比如 E:\\1.flv
#cap = cv2.VideoCapture(IPCameraUrl)  # 也可以打开IP摄像头
cap = cv2.VideoCapture(0)  # 0代表本地摄像头

cv2.namedWindow('video',cv2.WINDOW_NORMAL)

#打开摄像头

size = (int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)), int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)))

#画面尺寸  ffmpeg命令用
sizeStr = str(size[0]) + 'x' + str(size[1])


# windows 下要加 'cmd','/c', 不然会报错,linux下不加
command = ['cmd','/c',
    'ffmpeg',
    '-y', '-an',
    '-f', 'rawvideo',
    '-vcodec','rawvideo',
    '-pix_fmt', 'bgr24',
    '-s', sizeStr,
    '-r', '30',  # 帧率  尽量与原视频流帧率一致,否则会出现各种问题  broken pipe,速度慢等
    '-i', '-',
    '-c:v', 'libx264',
    '-pix_fmt', 'yuv420p',
    '-preset', 'ultrafast',
    '-f', 'flv',
    rtmp]

pipe = subprocess.Popen(command,stdin=subprocess.PIPE)

# 循环读取每一帧数据
while cap.isOpened():
    #返回标记和这一帧数据
    ret, frame = cap.read()
    if not ret:
        print("break")
        break 

    # 显示数据,用窗口显示图像,可能会造成延迟卡顿,一般不显示
    # cv2.imshow('video',frame)

    key = cv2.waitKey(1)
    # 窗口处输入q结束程序
    if key == ord('q'):
        break

    # 推流给服务器
    pipe.stdin.write(frame.tobytes())

pipe.terminate()
cap.release()
cv2.destroyAllWindows()
