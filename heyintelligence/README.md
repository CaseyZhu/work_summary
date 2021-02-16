# CNN Accelerator
This Accelerator is used to speed up the Convolutional Neural NetWork. 
It similar the pipe-line in ISP, each module do some specific work, 
and they can on/off as we need. The pipe-line as depicted in the figure below.<br>
![cnn-pipe](https://github.com/CaseyZhu/work_summary/blob/main/heyintelligence/image/cnna.jpg)
## Convolution Module
This module is designed to share data and weight between different lines and channels.
The main idea is borrowed from Eyeriss, 
and additional share of input feature-map between weight are added.<br> 
![](https://pic2.zhimg.com/v2-ce96d190df3f644fa645610be0b96e49_r.jpg)
## Partial Sum and Add bias
The convolution engine just compute part of the results, 
we need add all the results from the input channel.<br> 
![](https://pic3.zhimg.com/80/v2-16b8d1c25cd06bee7b0b32cf89b16bfa_720w.jpg)<br>
Add bias is the basic function in CNN.

## Elementwise 
Elementwise similar the vector operation, it add/sub/multiply elements in two feature map point by point.<br>
Elementwise's position may after bias,relu and pooling.

## Relu
Relu is a basic function in CNN. To support different kind of relu, 
this module set two threshold which divide the incomming data into three part.
each part have a coefficence and a bias. 
And per-channel relu also need to support, so these information need read backfrom external memory.
The structure as below.<br>
![relu](https://github.com/CaseyZhu/work_summary/blob/main/heyintelligence/image/relu.jpg)

## Pooling
The function of pooling is down sampling the input feature map. 
We support 3x3, 2x2 max/average pooling. 
Using the structure below we realize the data sharing between different pooling window.
The structure of pooling as below.<br> 
![pooling](https://github.com/CaseyZhu/work_summary/blob/main/heyintelligence/image/pooling.jpg)<br>
The special case is at the end of each line, the data is less than we need.
For example, 3x3 average pooling. 
1) at end there only left 6 point, we do not want to add additional divider, so use the formula below to reuse 1/9 function.<br>
```
sum/6 = (sum + 1/2sum) / 9
```
2) if only 4 point left, just repeat above operation two times.<br>
```
sum/4 = ((sum + 1/2sum) + 1/2(sum + 1/2sum)) / 9
```
3) The data is 8bits fix point data, max 9 points add together, 
so use 12 bits to store the sum is enough. And, the 1/9 function is realize by shift and add.
The formula as below:<br>
```
x/9 = x*[(2^12)/9]/(2^12) 
    = x*0x1c7 >> 12 
    = (x<<8 + x<<7 + x<<6 + x<<2 + x<<1 + x<<0) >> 12
```
use booth algorithm it can simplified as below.<br>
```
x/9 = [(x<<8 + x<<7 + x<<6) + (x<<2 + x<<1 + x<<0)] >> 12
    = [(x<<9 - x<<6) + (x<<3 - x<<0)] >> 12
```
## Softmax
The step of softmax as below:<br>
1) find top 5 max value in the icome data.
2) compute S, which is the sum of E = e^(x-max).
3) compute softmax R = E / S.
4) send out all the R and top 5's R.
To save gate count we use Taylor expansion to realize e^x and 1/x.<br>
```
e^x = e^(i+f) <small>where i is the integer part of x and f is the fraction part</small>
    = e^i * e^f
```
we use 8 bits fix point to save data, so e^i very easy to out of the representation range.
If out of range happened just use the max value represent it. 
And, we pre-compute the value of e^0.5, e^1, e^2,e^3,e^4, if i> 4 the results is max value.
e^f and 1/x can be realize by taylor expansion. Figure below depicted the realization.<br>
![ex](https://github.com/CaseyZhu/work_summary/blob/main/heyintelligence/image/ex.jpg)
![taylor](https://github.com/CaseyZhu/work_summary/blob/main/heyintelligence/image/taylor.jpg)

## Image Compression
Compression can reduce power and bandwidth. 
And, images have the character that the color is very similar in a small local area.
For example a 4x4 tile from the image may its pixel value like below.And find the smallest value in the 4x4 block, the  value is 251, all value in the  block minus 251, we obtain below block.
the max differential value is 4 , 3 bits is enough to store it. origin need 8 * 16 = 120 bits and now we can use 8 + 3 * 16 = 56  bits store it.  
The memory layout change to two part header and body. Header's size is fixed it store the address and size of body and the information to decompression the body.

|253|252|254|253|
|----|----|----|----|
|251|253|253|253|
|252|253|255|253|
|251|254|253|255|



|2|1|3|2|
|----|----|----|----|
|0|2|2|2|
|1|2|4|2|
|0|3|2|4|




