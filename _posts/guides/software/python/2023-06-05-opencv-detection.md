---
title: "OpenCV Face Detection"
excerpt: "Face detection with OpenCV"
toc: true
permalink: /guides/software/python/opencv-detection
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

## Finding a Classifier

OpenCV's face detection loads a [cascade classifier](https://docs.opencv.org/3.4/db/d28/tutorial_cascade_classifier.html) which is then used to detect faces.

Go to [github.com/opencv/opencv/data/haarcascades](https://github.com/opencv/opencv/tree/master/data/haarcascades). There are a number of different classifiers, download [haarcascade_frontalface_default.xml](https://github.com/opencv/opencv/blob/master/data/haarcascades/haarcascade_frontalface_default.xml)

## Running the classifier

Using an image with a face (or many faces); we can detect them with:
```python
import cv2 as cv

img = cv.imread('Photos/people.jpg')

cascade_file = 'haarcascade_frontalface_default.xml'
haar_cascade = cv.CascadeClassifier(cascade_file)
faces_rect = haar_cascade.detectMultiScale(
  cv.cvtColor(img, cv.COLOR_BGR2GRAY), 
  scaleFactor=1.1, minNeighbors=5)

for (x,y,w,h) in faces_rect:
    cv.rectangle(img, (x,y), (x+w, y+h), (0,255,255), 2)
cv.imshow('Detected faces', img)

cv.waitKey(0)
```

For the Super Mario Movie cast, I got these results.

<figure>
    <img src="/assets/images/posts/guides/opencv/00_detection.png"><img/>
</figure>
There were a few false positives and negatives. 

This could be further tuned by adjusting the `scaleFactor` and `minNeighbours` parameters; however, this isn't the best idea since we would only be tuning the face detection to work with a specific type of image.

