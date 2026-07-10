---
title: "OpenCV Training"
excerpt: "Training an OpenCV Face Recognition Model"
toc: true
permalink: /guides/software/python/opencv-recognition
categories:
  - guide
  - python
  - opencv
---

These are my notes based on Jason Dsouza's excellent [OpenCV video course](https://youtu.be/oXlwWbU8l2o).

## Installation

```
python3 -m pip install opencv-contrib-python
```

## Setup

We assume you have a the following folders:

```
train
|- Name A
|- Name B
|- ...etc
validate
|- Name A
|- Name B
|- ...etc
```

Each sub-folder should have multiple images in them for training and verifying.

Make sure you have downloaded the face detection cascade data from [opencv detection](/guides/software/python/opencv-detection).

## Training

```python
import cv2 as cv
import os
import numpy as np

DIR = "train"
people = []
for i in os.listdir(DIR):
    people.append(i)

cascade_file = 'haarcascade_frontalface_default.xml'
haar_cascade = cv.CascadeClassifier(cascade_file)
def detect_face(img):
  return haar_cascade.detectMultiScale(img, 
    scaleFactor=1.1, minNeighbors=4)

def create_train():
  features = []
  labels = []
  for person in people:
    path = os.path.join(DIR, person)
    label = people.index(person)

    for img in os.listdir(path):
      img_path = os.path.join(path, img)
      img_array = cv.imread(img_path)
      gray = cv.cvtColor(img_array, cv.COLOR_BGR2GRAY)

      for (x,y,w,h) in faces_rect:
        faces_roi = gray[y:y+h, x:x+w]
        features.append(faces_roi)
        labels.append(label)

  return np.array(features, dtype='object'), np.array(labels)

features, labels = create_train()

# Train the recognizer on the features list and labels list
face_recogniser = cv.face.LBPHFaceRecognizer_create()
face_recogniser.train(features, labels)
face_recogniser.save('face-trained.yml')
```

## Recognition

In this code, we use our trained face recognizer to determine if a detected face is who we expect it to be.

```python
import numpy as np
import cv2 as cv
import os

DIR = "train"
people = []
for i in os.listdir(DIR):
    people.append(i)

# Load model
face_recognizer = cv.face.LBPHFaceRecognizer_create()
face_recognizer.read('3-faces/face-trained.yml')

# Load a face
img = cv.imread('validate/elton_john/1.jpg')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

# Detect face
cascade_file = 'haarcascade_frontalface_default.xml'
haar_cascade = cv.CascadeClassifier(cascade_file)
faces_rect = haar_cascade.detectMultiScale(gray, 1.1, 4)
for (x,y,w,h) in faces_rect:
  faces_roi = gray[y:y+h, x:x+w]

  label, confidence = face_recognizer.predict(faces_roi)

  cv.putText(img, str(people[label]), (20,40), 
    cv.FONT_HERSHEY_COMPLEX, 1.0, (0, 255, 0), 2)
  cv.rectangle(img, (x,y), (x+w, y+h), (0,255,0), thickness=2)

cv.imshow('Detected face', img)

cv.waitKey(0)
```

This works sometimes from my poorly trained model.

<figure>
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_0.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_1.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_2.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_3.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_4.png" alt="">
</figure>

Funnily enough, it thought Jerry Seinfeld is Elton John when wearing glasses.
 
<figure>
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_5.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_6.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_7.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_8.png" alt="">
  <img style="width:20%" src="/assets/images/posts/guides/opencv/01_recognize_roi_9.png" alt="">
</figure>
