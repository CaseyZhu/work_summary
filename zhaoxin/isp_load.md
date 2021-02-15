# Load module
## Function Overview
The function of the load module is load raw data of images from external memory, 
and distributes the data into the line buffer. 
First, it need support different colour format, such as  ARGB8888,RGB565,YUV422,YUV420,etc.
Second, the data stored in external memory including two format linear and tile.
Third, it need some basic function of image process, such as image clip, cut slice, rotation and down scaling.<br>
Other factors need to take care:
1) the data returned from external may out of order.
2) external will retrurn more data than you want if the image data store in compression mode.
## Colour Format
### ARGB8888
| bits 31-24 | bits 23-16 | bits 15-8 | bits 7-0 |
|----|----|----|----|
|      A     |      R     |     G     |     B    |

### RGB565
| bits 15:11 |  bits 10-5 |  bits 4-0 |
|----|----|----|
|     R      |      G     |     B     |
### YUV422
| bits 31-24 | bits 23-16 | bits 15-8 | bits 7-0 |
|----|----|----|----|
|     V      |      Y     |     U     |     Y    |
### YUV420
in this format 4 Y share one UV, and Y, UV may store separately.
|  bits 7-0  |
|----|
|     Y      |

| bits 15-8  |  bits 7-0  |
|----|----|
|     U      |      V     |

## Data Layout in External memory
### Linear format 
In this format data store pixel by pixel untill one line done, and do a bus word alignment at the end of each line.
| l0   |     p0     |     p1     |     p2    |    p3    | p4 | ..... | pn |
|---- |----|----|----|----|----|----|----|
| l1   |     p0     |     p1     |     p2    |    p3    | p4 | ..... | pn |
```
......
```
| ln-1 |     p0     |     p1     |     p2    |    p3    | p4 | ..... | pn |
 |----|----|----|----|----|----|----|----|
|ln   |     p0     |     p1     |     p2    |    p3    | p4 | ..... | pn |
### Tile Format
In this format the neighbour pixels are store together. 
It more efficency than the linear format when the algorithm need multiple lines' data to process.
For example, if we want to do 4x4 pooling we can store 4x4 sub-image continuiously in the external memory, 
so only 512 bits data back we can start the process.
```
+----+----+----+----+           +------------+------------+-----------+----------+
| p0 | p1 | p2 | p3 |           |   p4*n+0   |   p4*n+1   |   p4*n+2  |  p4*n+3  |
+----+----+----+----+           +------------+------------+-----------+----------+
| p0 | p1 | p2 | p3 |           |   p4*n+0   |   p4*n+1   |   p4*n+2  |  p4*n+3  |
+----+----+----+----+ ......    +------------+------------+-----------+----------+
| p0 | p1 | p2 | p3 |           |   p4*n+0   |   p4*n+1   |   p4*n+2  |  p4*n+3  |
+----+----+----+----+           +------------+------------+-----------+----------+
| p0 | p1 | p2 | p3 |           |   p4*n+0   |   p4*n+1   |   p4*n+2  |  p4*n+3  |
+----+----+----+----+           +------------+------------+-----------+----------+    
```
## Basic Process
### Clip Image
In this mode we only want to process part of the image, the top, bottom,left, right may be cliped.
```
+ - - - - - - - - - +
| original image    |
|                   |
| +---------------+ |
| | clipped image | |
| +---------------+ |
|                   |
+ - - - - - - - - - +
```
### Cut Slice
Becaus line buffer's size is limited, so we need cut the image into slices when the iamge is too big.
There is an overlap region between different slices as below.
```
+ - - - - -*-+ - - - - -#-* - - - - --$-# - - - - -&-$ - - - - -- - - - &
| slice0+  | | slice1*  | | slice2#   | | slice3$  | | .....&           |
|          | |          | |           | |          | |                  |
|          | |          | |           | |          | |                  |
|          | |          | |           | |          | |                  |
|          | |          | |           | |          | |                  |
|          | |          | |           | |          | |                  |
+ - - - - -*-+ - - - - -#-* - - - - --$-# - - - - -&-$ - - - - -- - - - &
```
### Rotation
In this mode we need rotate the image.
```
+ - - - - - - - - - +  + - - - - - - - - - + + - - - - - - - - - + + - - - - - - - - - +
| original image    |  |  rotate 90        | |  rotate 180       | | rotate 270        |
|                   |  |       .           | |                   | |       #           |
|                   |  |       .           | |                   | |       .           |
|     .......#      |  |       .           | | #.......          | |       .           |
|                   |  |       .           | |                   | |       .           |
|                   |  |       #           | |                   | |       .           |
+ - - - - - - - - - +  + - - - - - - - - - + + - - - - - - - - - + + - - - - - - - - - +
```
### Down Scaling
Down scaling 2x 4x in both x,y direction
## Data swizzle in line buffer's bank
There are 8 banks of sram in line buffer, the data width of the bank is 32 bits. 
Different lines start at different banks like below. Bank number = y[2:0] + x[2:0]
```
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |     bank0     |  |   bank1    ||   bank2   ||  bank3   || bank4 || bank5 || bank6 || bank7 |
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p0       |  |     p1     ||    p2     ||    p3    ||  p4   ||  p5   ||  p6   ||  p7   |
line0 +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p8       |  |     p9     ||    p10    ||   p11    ||  p12  ||  p13  ||  p14  ||  p15  |
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      ......
      ...
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p7       |  |     p0     ||    p1     ||    p2    ||  p3   ||  p4   ||  p5   ||  p6   |
line1 +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p15      |  |     p8     ||    p9     ||   p10    ||  p11  ||  p12  ||  p13  ||  p14  |
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      ......
      ...
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p6       |  |     p7     ||    p0     ||    p1    ||  p2   ||  p3   ||  p4   ||  p5   |
line2 +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      |      p14      |  |    p15     ||    p8     ||    p9    ||  p10  ||  p11  ||  p12  ||  p13  |
      +---------------+  +------------++-----------++----------++-------++-------++-------++-------+
      ......
      ...
``` 
## Data back out of order
During send out the requests, 
we have compute out the bank number and banks address and put these information into tag.
When data back we can easily obtain the bank number and banks address.
