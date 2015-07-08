from pylab import *
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.cbook as cbook
import random
import time
from scipy.misc import imread
from scipy.misc import imresize
import matplotlib.image as mpimg
import os

# Please change this to the directory with the given images for the program 
# to work correctly
os.chdir('###')

def cropborders(image):
    height = image.shape[0]
    length = image.shape[1]
    
    cropped_image = image[height*.05:height-height*.03,length*.05:length-length*.07]
    
    return cropped_image


def compare(image1, image2, type):
    pushx = 0
    pushy = 0
    if type == 'ncc':
        lower = 0
        upper = 0
    else:
        upper = inf
        lower = 0
    
    for i in range(-10, 11):
        temp = roll(image2, i, 0)
        for j in range(-10, 11):
            if type == 'ncc':
                upper = sum((image1 - mean(image1))*(roll(temp, j, 1) - \
            mean(roll(temp, j, 1))) ) / ( norm(image1 - mean(image1)) *\
             norm(roll(temp, j, 1) - mean(roll(temp, j, 1))))
            else:
                lower = np.sum((image1-roll(temp, j, 1))**2)
            if lower < abs(upper):
                if type == 'ncc':
                    lower = abs(upper)
                    value = abs(upper)
                else:
                    upper = lower
                    value = lower
                pushx = i
                pushy = j
                
    return [value, pushx, pushy]
    
    
def separate_and_combine(image, type):
    height = image.shape[0]/3
    
    blue = image[:height,]
    green = image[height + 1: 2*height + 1,]
    red = image[2*height + 1:,]
    
    
    
    blue = cropborders(blue)
    green = cropborders(green)
    red = cropborders(red)
    
    
    
    size = min(blue.shape[0], red.shape[0], green.shape[0])
    
    
    # Ensures all dimensions are the same as the smallest segment
    if blue.shape[0] - size > 0:
        blue = blue[blue.shape[0] - size:]
    if green.shape[0] - size > 0:
        green = green[green.shape[0] - size:]    
    if red.shape[0] - size > 0:
        red = red[red.shape[0] - size:]
    
    channels = [red, green, blue]
            
    if green.shape[1] > 3000:
        return large_image(channels, type)
    
    red = compare_and_shift(green, red, type, False)
    blue = compare_and_shift(green, blue, type, False)

    rgb = zeros([green.shape[0], green.shape[1], 3])
    rgb[:,:,0] = red
    rgb[:,:,1] = green
    rgb[:,:,2] = blue
    rgb = 255 - rgb
        
    return rgb
   
   
def large_image(channels, type):
    small_red = imresize(channels[0], 10)   
    small_green = imresize(channels[1], 10)
    small_blue = imresize(channels[2], 10)
    
    for channel in channels:
        print(channel.shape[0])
    
    red_shift = compare_and_shift(small_green, small_red, type, True)
    blue_shift = compare_and_shift(small_green, small_blue, type, True)
    
    print([red_shift, blue_shift])
    channels[0] = roll(channels[0], red_shift[0], 0)
    channels[0] = roll(channels[0], red_shift[1], 1)
    
    channels[2] = roll(channels[2], blue_shift[0], 0)
    channels[2] = roll(channels[2], blue_shift[1], 1)
    
    rgb = zeros([channels[1].shape[0], channels[1].shape[1], 3])
    rgb[:,:,0] = channels[0]
    rgb[:,:,1] = channels[1]
    rgb[:,:,2] = channels[2]
    rgb = 255 - rgb
    
    return rgb
    
   
def compare_and_shift(image1, image2, type, big):
    # images will be same dimensions so only need 
    # this once    
    length = image1.shape[1]/5
    height = image1.shape[0]/5    
    
    if type == 'ncc':
        # NCC values are fairly constant across all segments of the picture, so 
        # the value is taken only once from the inside of the photo
        best_value = compare(image1[height:5*height - height,length:\
            5*length - length], image2[height:5*height - height,length:5*\
            length - length], 'ncc')  
    else:
        # Sample a variety of areas to find the closest match
        values1 = compare(image1[:,length:2*length], image2[:,length:2*length], 'ssd')
        values2 = compare(image1[:,2*length:3*length], image2[:,2*length:3*length],'ssd')
        values3 = compare(image1[:,3*length:4*length], image2[:,3*length:4*length],'ssd')
        values4 = compare(image1, image2,'ssd')
        closest = min([values1[0], values2[0], values3[0], values4[0]])
        for set in [values1, values2, values3, values4]:
        # Find the displacement corresponding to the closest match
            if closest in set:
                best_value = set
                print((set, '*'))
    
    pushx = best_value[1]
    pushy = best_value[2]
    
    if big:
        return [pushx, pushy]
    
    image2 = roll(image2, pushx, 0)
    image2 = roll(image2, pushy, 1)
    
    return image2
    
    
def main():
    image1 = imread('01880v.jpg')
    # Both techniques create the same final image 
    ncc = separate_and_combine(image1, 'ncc')
    ssd = separate_and_combine(image1, 'ssd')
    figure(1); imshow(ncc)
    figure(2); imshow(ssd)
    
    image2 = imread('01657v.jpg')
    # This one is the best match using ncc, but using ssd the match is poor
    ncc2 = separate_and_combine(image2, 'ncc')
    ssd2 = separate_and_combine(image2, 'ssd')
    figure(3); imshow(ncc2)
    figure(4); imshow(ssd2)
    
if __name__ != "__main__":
    main()
    
    
    
