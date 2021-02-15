# Basic Flow of VPP/ISP
The basic flow of vpp(video post process)/isp as fellows: <br>
1) Load module load data from external memory, and store the data into line buffer.<br>
2) The communication between different modules using frame id and x,y coordinate.<br>
3) The module below load will take read data from line buffer if there are enough data for them. 
Load module will continue load data from external memory if others release the space of line buffer.<br>
4) Some module can only obtain data from pipe-line.<br>
5) Finally, the processed data will be send back to external memory through Store module.<br> 

```
                           +--------------+
                           | external mem |
                           +--------------+
                             |
  +--------------+           |
  |              |           v
  |              |       +- - - - - - - - - +
  |              |       ' pipe-line:       '
  v              |       '                  '
+-------------+  |       ' +--------------+ '
| line buffer |  +------ ' |     load     | '
+-------------+  |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 +------ ' |     dnt      | '
                 |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 |       ' |     dns      | '
                 |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 +------ ' |     csc      | '
                 |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 |       ' |    sharp     | '
                 |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 |       ' |   scaling    | '
                 |       ' +--------------+ '
                 |       '   ^              '
                 |       '   |              '
                 |       '   |              '
                 |       '   v              '
                 |       ' +--------------+ '
                 +------ ' |    store     | '
                         ' +--------------+ '
                         '                  '
                         +- - - - - - - - - +
                             |
                             |
                             v
                           +--------------+
                           | externalmem  |
                           +--------------+
```
