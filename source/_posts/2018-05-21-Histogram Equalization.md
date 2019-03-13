---
layout: post
title:  "Histogram Equalization"
date:   2018-05-21
categories: DIP
---

## Histogram Equalization

- Histogram equalization is used to enhance contrast.

- Lets start histogram equalization by taking this image below as a simple image.

- The histogram of this image has been shown below.

  ![](/image/hisE01.png)

  ![](/image/hisE02.png)

- Now we will perform histogram equalization to it.

- First we have to calculate the PMF (probability mass function) of all the pixels in this image.

- Our next step involves calculation of CDF (cumulative distributive function).  

## Calculate CDF according to gray levels

![](/image/hisE03.png)

- Then in this step you will multiply the CDF value with (Gray levels (minus) 1) .

- Considering we have an 3 bpp image. 

- Then number of levels we have are 8.

- And 1 subtracts 8 is 7. So we multiply CDF by 7. Here what we got after multiplying.

  ![](/image/hisE04.png)

- we have to map the new gray level values into number of pixels.

- Lets assume our old gray levels values has these number of pixels.

![](/image/hisE05.png)

- Now if we map our new values to , then this is what we got.

  ![](/image/hisE06.png)

- Now map these new values you are onto histogram, and you are done.

- Lets apply this technique to our original image. 

- After applying we got the following image and its following histogram.

## Cumulative Distributive function of this image

![](/image/hisE07.png)

## Histogram Equalization histogram

![](/image/hisE08.png)

## Comparing both the histograms and images

![](/image/hisE09.png)

## Conclusion

- during histogram equalization the overall shape of the histogram changes, 
- where as in histogram stretching the overall shape of histogram remains same.

 

