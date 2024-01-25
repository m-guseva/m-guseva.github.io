---
title: "👀 Algorithm that determines homogeneous areas in images (Python)"
excerpt: "This algorithm finds those areas in an image that are similar in value. <br/><img src='https://github.com/m-guseva/m-guseva.github.io/blob/master/images/thumb_imageAlgorithm.jpg?raw=true'>"
collection: portfolio
---
## The issue
For my "value vision training" project (more on that here) I faced a challenge: I needed a way to select those areas in an image  where the surrounding portions of the image were homogenous in terms of tonal value. See example below:

![ImageAlgorithm_1](https://github.com/m-guseva/personal/assets/63409978/1f5cb2ae-bf40-47f1-997f-62791853ef7a)

I had a lot of fun developing the algorithm, taking it from idea to actually working like it I envisioned it 😎.

## Step-by-step
![ImageAlgorithm_2](https://github.com/m-guseva/personal/assets/63409978/c561aaeb-7b25-4c79-906b-1bc2cc41e283)


(1) Randomly select a pixel in the image (let's call it targetPixel, yellow in the image). This pixel cannot be close to the borders of the image, otherwise we cannot construct a square around it, so a permissible zone by "padding" the image based on radius $r$ is created.

(2) Select surrounding pixels with radius $r$ (blue area in image)

(3) Calculate the tonal value of targetPixel and the average of all surrounding neighbour pixels.

(4) Take the absolute difference between the targetPixel value and average neighbour pixel value

(5) If this difference is larger than some small number $epsilon$, then find another pixel (go back to step 1). If it is equal or smaller than $epsilon$, then we found a homogenous area 🏅!

In the context of the vision trainer, this targetpPixel is selected and a box is drawn around it. Then the user tries to guess the tonal value by means of a slider that ranges from 0 (black) to 100 (white).

## The code
This is the algorithm:
### Import libraries
```python
import pandas as pd
from PIL import Image, ImageDraw, ImageOps
import numpy as np
import math
from functions import *
```
### Image management
As an example, I'm importing an image using Python's PIL library. The images is converted to greyscale and then to 'L' (luminance) mode where each pixel carries the luminance (tonal value) information (instead of the color information).

```python
# open image
image = Image.open('/image.png')
# convert to grayscale
image = ImageOps.grayscale(image) 
# draw image on canvas
draw = ImageDraw.Draw(image)
# Convert to luminance values (ranging from 0=black to 255=white)
bw_image = array(image.convert("L")) 
```

### Define permissible values
```python

def permissibleValues(bw_image,perc):
    """returns permissible values for row and columns to use later. If the values are
    too close to the edges, then the we can't plot a full square. That's why we need to
    pad the image with a radius. 
    Parameters: 
    - perc: how much percent of either width or height do we take
    - bw_image: bw_image object"""
    radius=int(max(bw_image.shape)*perc) # % of largest of width or height
    permissibleCol = np.arange(0+radius, bw_image.shape[0]-radius)
    permissibleRow = np.arange(0+radius, bw_image.shape[1]-radius)
    return permissibleCol, permissibleRow, radius


permissibleCol, permissibleRow, radius = permissibleValues(bw_image,0.02)
```

### Extract surrounding neighbours, calculate tonal values and compare difference to epsilon
```python
def nearestNeighbours(i,j,radius,bw_image):
    '''Function takes permissible i and j positions of a target pixel and outputs neighboring values 
    
    Parameters:
    i:column
    j:row 
    radius:radius around target pixel'''
    cols = np.ndarray.tolist(np.arange(i-radius,i+1+radius))
    rows = np.ndarray.tolist(np.arange(j-radius,j+1+radius))
    neighbours = []
    for x in range(len(cols)):
        for y in range(len(rows)):
            if (cols[x] == i) & (rows[y]== j):
                pass
            else:
                neighbours.append(bw_image[cols[x], rows[y]])
    return neighbours


diff = 1+epsilon #initialize a value
while(diff>epsilon):    
    # Sample random elements from the permissible x and y coordinates of the image to select a valid random targetPixel.
    targetX = np.random.choice(permissibleCol)
    targetY = np.random.choice(permissibleRow)

    # Get nearest neighbours surrounding targetPixel
    nnVals = nearestNeighbours(targetX, targetY,radius,bw_image)

    # Calculate absolute difference between tonal value of target pixel and mean tonal value of neighbour pixels 
    diff = abs(bw_image[targetX, targetY]-np.mean(nnVals))
```
