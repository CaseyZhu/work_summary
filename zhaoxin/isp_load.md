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
+----+----+----+----+           + ------------+------------+-----------+----------+
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
![clip](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/clip.svg ){:height="200px" width="200px"}
### Cut Slice
Becaus line buffer's size is limited, so we need cut the image into slices when the iamge is too big.
There is an overlap region between different slices as below.
![slice](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/cut_slice.svg = 200x200)
### Rotation
In this mode we need rotate the image.
![rotate0](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/rotate0.jpg = 200x200) ![rotate1](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/rotate1.svgz = 200x200)
### Down Scaling
Down scaling 2x 4x in both x,y direction
![](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/ds24.jpg = 200x200) ![](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/ds24.jpg = 100x100) ![](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/ds24.jpg = 50x50)
## Data swizzle in line buffer's bank
There are 8 banks of sram in line buffer, the data width of the bank is 64 bits. 
Different lines start at different banks like below. Bank number = y[2:0] + x[3:1]
![](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/swizzle.jpg = 200x200) 
## Data back out of order
During send out the requests, 
we have compute out the bank number and banks address and put these information into tag.
When data back we can easily obtain the bank number and banks address.
## Multi-Frame
DNT need three frame. We need load two more frame back if DNT on. 
We obtain other frame's address by adding an offset to current frame's address. 
