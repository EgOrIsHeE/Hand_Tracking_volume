import cv2
import time
import numpy as np
import HandTrackingModule as htm
import math
from ctypes import cast, POINTER
from comtypes import CLSCTX_ALL
from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume

wCam, hCam = 640, 480
cap = cv2.VideoCapture(0)  # провеярем, работает ли камера
cap.set(3, wCam)  # устанавливаем длину диалогового окна
cap.set(4, hCam)  # устанавливаем ширину диалогового окна
pTime = 0
detector = htm.handDetector(detectionCon=0.7)  # класс из модуля с уже описанными методами(смотри handtrackingmin)
devices = AudioUtilities.GetSpeakers()
interface = devices.Activate(IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
volume = cast(interface, POINTER(IAudioEndpointVolume))
volRange = volume.GetVolumeRange()  # получаем диапазон звука
minVol = volRange[0]
maxVol = volRange[1]
vol = 0
volPer = 0
while True:
    success, img = cap.read()
    img = detector.findHands(img)
    lmList = detector.findPos(img, draw=False)
    if len(lmList) != 0:
        x1, y1 = lmList[4][1], lmList[4][2]  # координаты четвертой точки для руки
        x2, y2 = lmList[8][1], lmList[8][2]  # координаты восьмой точки для руки
        cx, cy = (x1 + x2) // 2, (y1 + y2) // 2  # координаты центра линии, соединяющей точки
        cv2.circle(img, (x1, y1), 15, (255, 0, 255), cv2.FILLED)  # выделяем необходимые точки другим цветом и делаем их пожирнее
        cv2.circle(img, (x2, y2), 15, (255, 0, 255), cv2.FILLED)
        cv2.line(img, (x1, y1), (x2, y2), (255, 0, 255), 3)  # проводим линию между точками
        cv2.circle(img, (cx, cy), 15, (255, 0, 255), cv2.FILLED)  # выделяем центр линии
        length = math.hypot(x2 - x1, y2 - y1)  # длина линии в пикселях
        vol = np.interp(length, [50, 300], [minVol, maxVol])  # переводим диапазон звука в диапазон длины
        volPer = np.interp(length, [50, 300], [0, 100])  # переводим звук в проценты
        print(int(length), vol)
        volume.SetMasterVolumeLevel(vol, None)
        if length < 50:
            cv2.circle(img, (cx, cy), 15, (0, 255, 0), cv2.FILLED)  # меняем цвет шарика посередине, если звук стал меньше опр. значения

    cv2.putText(img, f'{int(volPer)} %', (40, 450), cv2.FONT_HERSHEY_COMPLEX, 1, (255, 0, 0), 3)  # выводим громкость в процентах
    cTime = time.time()
    fps = 1 / (cTime - pTime)
    pTime = cTime
    cv2.putText(img, f'FPS: {int(fps)}', (40, 50), cv2.FONT_HERSHEY_COMPLEX, 1, (255, 0, 0), 3)  # выводим FPS

    cv2.imshow("Img", img)#выводим изображение с вебки на экран компа
    cv2.waitKey(1)
