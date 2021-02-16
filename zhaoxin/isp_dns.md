# DNS module
DNS(Denoise in space domain), there are three sub modules: SBPC,DBPC,DNS. 
They work together to realize the denoise function.
## SBPC
Static bad pixel correction's function is to identify the defects caused by the sensors.
The bad pixel's coordinates are pre-stored in the external memory, 
we need read them back and store into loacal buffer.
Bad pixel flag will set  if the coming pixels' coordinate match static pixel's coordinates.
The flow of sbpc as below:<br>
![sbpc_flow](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/sbpc_flow.jpg)
## DBPC
Dynamic bad pixel correction's function is to identify bad/weak pixels accroding to the surrounding pixels.
2 bits flag used to sign the pixel as good, bad and weak pixel. 
The principle is that if the pixel's value is not nearby its surrounding, it may a bad pixel.<br>
![dbpc](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/dbpc.jpg)
## Bad Pixel Recover
If the pixel is not a good one, we need recover it. The easist way is use mean method.<br>
![bpc_mean](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/recover_mean.jpg)
## DNS
DNS is a complicated algorithm, it need a 11x11 window, so in HW it need a long pipe-line to finish this.<br>
![dns](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/dns_flow.jpg)
## Summary
1) DNS SBPC DBPC can enable independently.
2) If pixel is bad the ouput value will replaced by recover value.
3) DNS SBPC DBPC need different window size, HW need make synchronization between them.

