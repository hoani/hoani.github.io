---
title: "OpenCV Basics"
excerpt: "Basic operations in OpenCV with Python"
toc: true
permalink: /guides/software/python/opencv
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

## Showing an Image

```py
import cv2 as cv

img = cv.imread('Photos/profile.jpg')
cv.imshow('Profile', img)
cv.waitKey(0) # Waits forever until keypress.
```

## Showing a Video

Showing a video is similar to an image, but we display each frame individually.

```py
import cv2 as cv

# capture = cv.VideoCapture(0) for webcam
capture = cv.VideoCapture('Videos/drone.mp4')

while True:
  hasFrame, frame = capture.read()
  if not hasFrame:
    break
  cv.imshow('Video', frame)
  if cv.waitKey(20) & 0xFF==ord('d'):
    break

capture.release()
cv.destroyAllWindows()
```



## Common Operations

For each example, assume we start with:
```python
import cv2 as cv

img = cv.imread('Photos/profile.jpg')
```

### Convert to Grayscale

```python
gray = cv.cvtColor(img, code=cv.COLOR_BGR2GRAY)
cv.imshow('Gray', gray)
```

### Blurring

```python
blur = cv.GaussianBlur(img, (7,7), cv.BORDER_DEFAULT)
cv.imshow('Gray', gray)
```

### Edge Cascade

```python
canny = cv.Canny(blur, 125, 175)
cv.imshow('Canny Edges', canny)
```

_Note: Blurring prior to edge detection can provide better results._

### Dilating/Eroding

```python
dilated = cv.dilate(canny, (7,7), iterations=3)
eroded = cv.erode(dilated, (7,7), iterations=3)
cv.imshow('Dilated', dilated)
cv.imshow('Eroded', eroded)
```

## Drawing on an image

For each example, assume we start with:
```python
import cv2 as cv

img = np.zeros((500, 500, 3), dtype='uint8')
green = 0,255,0
red = 0,0,255 # Note BGR.
```

### Painting pixels
```python
blank[:] = green
blank[200:300, 300:400] = red
cv.imshow('Green with red square', img)
```

### Rectangle

```python
cv.rectangle(img, (0, 0), (250, 400), green, thickness=cv.FILLED)
cv.imshow('Rectangle', img)
```

Set thickness to an integer for outlines.

### Circle
```python
cv.circle(img, (250, 250), 100, red, thickness=cv.FILLED)
cv.imshow('Circle', img)
```

### Line
```python
cv.line(img, (50, 200), (450, 300), green, thickness=4)
cv.imshow('Line', img)
```

### Text
```python
cv.putText(img, "Hello", (225, 225), cv.FONT_HERSHEY_TRIPLEX, 1.0, green, thickness=2)
cv.putText(img, "Bye bye", (225, 280), cv.FONT_HERSHEY_DUPLEX, 1.0, green, thickness=2)
cv.imshow('Text', img)

cv.waitKey(0)
```

## Transformations

For each example, assume we start with:
```python
import cv2 as cv
import numpy as np

img = cv.imread('Photos/profile.jpg')
```

### Translation

```python
def translate(img, x, y):
    transMat = np.float32([[1,0,x], [0,1,y]])
    dimensions = (img.shape[1], img.shape[0])
    return cv.warpAffine(img, transMat, dimensions)

translated = translate(img, -100, 100)
cv.imshow('Translated', translated)
```

### Rotation

```python
def rotate(img, angle, rotPoint=None):
    (h, w) = img.shape[:2]
    if rotPoint is None:
        rotPoint = (w//2, h//2)
    
    rotMat = cv.getRotationMatrix2D(rotPoint, angle, 1.0)
    dimensions = (w, h)

    return cv.warpAffine(img, rotMat, dimensions)

rotated = rotate(img, -45)
cv.imshow('Rotated', rotated)
```

### Resizing

```python
resized = cv.resize(img, (500,500), interpolation=cv.INTER_NEAREST)
cv.imshow('Resized', resized)
```

_Note: Pay attention to interpolation; linear or cubic looks the best when increasing image sizes._

### Cropping

```python
cropped = img[50:200, 200:300]
cv.imshow('Cropped', cropped)
```

### Flipping

```python
cv.imshow('Flip Vertical', cv.flip(img, 0))
cv.imshow('Flip Horizontal', cv.flip(img, 1))
cv.imshow('Flip Both', cv.flip(img, -1))
```

## Contours

```python
import cv2 as cv
import numpy as np

img = cv.imread('Photos/profile.jpg')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

ret, thres = cv.threshold(gray, 125, 255, cv.THRESH_BINARY)
cv.imshow('Threshold', thres)

# RETR_EXTERNAL provides only external contours
# CHAIN_APPROX_SIMPLE allows us to combine points when it 
#   makes sense
#   i.e. (0,0), (1,1), (2,2) => (0,0), (2,2)
contours, heirachies = cv.findContours(thres, cv.RETR_LIST, cv.CHAIN_APPROX_NONE)
print(f'{len(contours)} contour(s) found')

blank = np.zeros(img.shape, dtype='uint8')
cv.drawContours(blank, contours, -1, (0,0,255), 1)
cv.imshow('Contours drawn', blank)

cv.waitKey(0)
```

The heirachies allow us to detect contours within contours, which may be useful for detecting features within objects.
