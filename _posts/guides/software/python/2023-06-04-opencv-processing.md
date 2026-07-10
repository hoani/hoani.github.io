---
title: "OpenCV Processing"
excerpt: "Image processing with OpenCV in python"
toc: true
permalink: /guides/software/python/opencv-processing
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

## Colorspaces

OpenCV represents pixels in a BGR (blue, green, red) colorspace, use `cv.cvtColor` when you need to represent pixels in other colorspaces.

```python
import cv2 as cv
import matplotlib.pyplot as plt

img = cv.imread('Photos/profile.jpg')
cv.imshow('Profile', img)

# Matplotlib uses RGB
plt.imshow(cv.cvtColor(img, cv.COLOR_BGR2RGB))
plt.show()
```

### Split/Merge

Colors can be split into each individual channel.

```python
import cv2 as cv
import numpy as np

img = cv.imread('Photos/profile.jpg')
cv.imshow('Park', img)

b, g, r = cv.split(img)

# Show colors for split images
blank = np.zeros(img.shape[:2], dtype='uint8')
cv.imshow('Blue', cv.merge([b, blank, blank]))
cv.imshow('Green', cv.merge([blank, g, blank]))
cv.imshow('Red', cv.merge([blank, blank, r]))

merged = cv.merge([b, g, r])
cv.imshow('Merged', merged)

cv.waitKey(0)
```

## Smoothing

Typically, we use smoothing to remove noise.

Smoothing functions use a kernal size which defines the local pixels which are used for the smoothing operation.

For each example, assume we start with:
```python
import cv2 as cv

img = cv.imread('Photos/profile.jpg')
```

### Averaging
Each pixel is the average of the neighbouring pixels.

```python
average = cv.blur(img, [11,11])
cv.imshow('AverageBlur', average)
```

### Gaussian
Each pixel is the gaussian weighted average of the neighbouring pixels.

```python
gaussian = cv.GaussianBlur(img, [11,11], sigmaX=0)
cv.imshow('GaussianBlur', gaussian)
```

### Median Blur
Each pixel is the median of the neighbouring pixels.

```python
median = cv.medianBlur(img, 11)
cv.imshow('MedianBlur', median)
```

### Bilateral Blur
Applies a bilateral blur which provides smooth transitions.

```python
bilateral = cv.bilateralFilter(img, 11, 35, 25)
cv.imshow('Bilateral', bilateral)
```

## Bitwise Operations

Can apply AND, OR, NOT and XOR. 

```python
import cv2 as cv
import numpy as np

blank = np.zeros((400, 400), dtype='uint8')
rectangle = cv.rectangle(blank.copy(), (30, 30), (370, 370), 255, -1)
circle = cv.circle(blank.copy(), (200, 200), 200, 255, -1)

bitwiseAnd = cv.bitwise_and(rectangle, circle)
cv.imshow('Bitwise AND', bitwiseAnd)

bitwiseOr = cv.bitwise_or(rectangle, circle)
cv.imshow('Bitwise OR', bitwiseOr)

bitwiseXor = cv.bitwise_xor(rectangle, circle)
cv.imshow('Bitwise XOR', bitwiseXor)

bitwiseNot = cv.bitwise_not(rectangle)
cv.imshow('Bitwise NOT', bitwiseNot)

cv.waitKey(0)
```

### Masking

Can use bitwise functions to mask an image.

```python
import cv2 as cv
import numpy as np

img = cv.imread('Photos/profile.jpg')

mask = cv.circle(np.zeros(img.shape[:2], dtype='uint8'), 
  (img.shape[1]//2, img.shape[0]//2), img.shape[0]//3, 
  255, -1)

masked = cv.bitwise_or(img, img, mask=mask)
cv.imshow('Masked image', masked)

cv.waitKey(0)
```

## Histogram

```python
import cv2 as cv
import matplotlib.pyplot as plt
import numpy as np

img = cv.imread('Photos/profile.jpg')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

gray_hist = cv.calcHist([gray], [0], None, [256], [0,256])
fig = plt.figure()
plt.title('Grayscale histogram')
plt.xlabel('Bins')
plt.ylabel('Num pixels')
plt.plot(gray_hist)
plt.xlim([0,256])
plt.show()
```

## Thresholding

Thresholding allows us to process an image based on pixel intensity.

For each example, assume we start with:
```python
import cv2 as cv

img = cv.imread('Photos/profile.jpg')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
```

### Simple

```python
th, thresh = cv.threshold(gray, 150, 255, cv.THRESH_BINARY)
cv.imshow('Simple Threshold', thresh)
```

### Inverted

```python
th, thresh = cv.threshold(gray, 150, 255, cv.THRESH_BINARY_INV)
cv.imshow('Inverted Threshold', thresh)
```

### Adaptive

```python
thresh = cv.adaptiveThreshold(gray, 255, cv.ADAPTIVE_THRESH_MEAN_C, cv.THRESH_BINARY, 11, 3)
cv.imshow('Adaptive threshold', thresh)
```

## Gradient Detection

Gradient detection is similar to edge detection; and is useful for simple object detection.

For each example, assume we start with:
```python
import cv2 as cv
import numpy as np

img = cv.imread('Photos/park.jpg')
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
```

### Lapacian
```python
lap = cv.Laplacian(gray, cv.CV_64F)
lap = np.uint8(np.absolute(lap))
cv.imshow('Laplacian', lap)
```

### Sobel
```python
sobelx = cv.Sobel(gray, cv.CV_64F, 1, 0)
sobely = cv.Sobel(gray, cv.CV_64F, 0, 1)
combined_sobel = cv.bitwise_or(sobelx, sobely)
cv.imshow('Sobel Combined', combined_sobel)
```

### Canny
```python
canny = cv.Canny(gray, 150, 175)
cv.imshow('Canny', canny)
```
