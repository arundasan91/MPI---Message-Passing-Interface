# Scatter and Gather back images using MPI.
<b> Images can be quite large sometimes and handling them can be computationaly intensive. So is videos. The following tutorial will focus on the power of MPI and OpenCV to significantly speed up the processing time required while handling images. </b>

So in Spring 2016, I took a cloud computing + parallel programming course which introduced me to MPI. One of the projects was to parallelize a program which counted the number of stars in a high detail, high pixel density image of the Andromeda Galaxy, a huge JPEG file. One of the task was to find the areas of concern and the areas where the program is actually parallelizable. The other was to get the actual number of stars counted as close as the values from NASA.

Size of the image file is 1.7GB and you can download the image from [here.](http://www.spacetelescope.org/static/archives/images/publicationtiff40k/heic1502a.tif)

Amdahls law gives us serious insight to the parallel performance of these programs. I started off with the serial code, carefully checking for key areas of interest. After finding out good candidates, I used the power of MPI to parallelize the code across different nodes.

I will try to introduce the basic concepts in other tutorials. Here, let me jump into what I have done to parallelize the code. The code was efficient enough to land the first place (execution time wise) comparing with the rest of the class. Note: The serial code was developed by a friend of ours who took the class the previous semester. 

Let us start.
Once you have installed MPI4PY and OpenCV, in a python shell, import the following packages.
```
import numpy as np		# IMPORT 'numpy' LIBRARY AS 'np'
import cv2			# IMPORT 'cv2' LIBRARY TO WORK WITH IMAGES
from mpi4py import MPI		# IMPORT 'MPI' FROM 'mpi4py' LIBRARY
import time			# IMPORT 'time' MODULE
```

Any MPI problems will have a communicator defined. It is a set of processes that help run the MPI code by communicating to each other. Here, we define a communicator `comm`. The `size` and `rank` of the communicator are also defined. 
```
comm = MPI.COMM_WORLD		# DEFINE THE COMMUNICATOR
size = comm.Get_size()		# FIND THE SIZE OF THE COMMUNICATOR CLASS
rank = comm.Get_rank()		# FIND THE RANK OF THE CURRENT NODE
```

The image file is huge. Our final goal is to count the number of stars in the image as precise as possible in the least amount of time. Here is the plan: 
1.	Read the downloaded image to one node.
2.	Break the image into smaller chunks.
3.	Broadcast the local image dimensions to each node.
4.	Scatter each part of image to different nodes.
5.	Count the number of stars in each node.
6.	Reduce the star-count back using sum operation.
7.	Display the result.

Step 1, 2: Download the image to one node. Let us call it the master node and use `rank` as zero. So only in `rank == 0` read the downloaded image, log time required to read the image. Another timer is stated to find the total time required for the code to execute. The dimensions of the whole image is found out and local dimensions of the image which will be scattered to other nodes are computed. In every case where `rank != 0`, we define the variables as `None`. The variables need to be defined `None` in-order to prevent errors during broadcast and scatter steps in nodes other than master node. The datatype used to define image files is `uint8`.
```
if rank == 0:			#LOAD THE IMAGE ONLY IN THE MASTER NODE
	t1 = MPI.Wtime()	# FIND THE TIME REQUIRED TO LOAD THE IMAGE
#	img = cv2.imread('heic1502a.tif',0)	# LOAD THE IMAGE 
	img = cv2.imread('galaxy.jpg',0)
	t2 = MPI.Wtime()
	print "Time required to read the image is ", t2-t1	# PRINT THE TIME REQUIRED TO LOAD THE IMAGE
#	print "Shape of image is ",img.shape[0],'x',img.shape[1]
	no_rows = int(img.shape[0])	# FIND THE IMAGE DIMENSIONS
	no_cols = int(img.shape[1])
	local_r = int(no_rows/size)	# DEFINE LOCAL DIMENSIONS
	local_c = int(no_cols/size)
	local_x = np.empty((no_rows, local_c),dtype='uint8')	# DEFINE LOCAL IMAGE TO BE SCATTERED TO EACH NODE
	Inarray = np.array([no_rows, local_c])			# DEFINE LOCAL IMAGE TO BE SCATTERED TO EACH NODE
else:	
	newimg 	= None	# DEFINE DEFAULT 'None' VALUES FOR VARIABLES
	no_rows = None
	img 	= None
	local_c = None
	Inarray = None
```
Step 3, 4: The master node is now set-up. Now, we need to run the `bcast` command in each and every node to broadcast the local image dimensions and then the `Scatterv` command to scatter different chunks of images to different nodes. Note that we use `Scatterv` here to allow irregular image dimensions while scattering. We compute our local image dimensions as `local_r = int(no_rows/size)` and `local_c = int(no_cols/size)` which may not always divide the image equally. So `Scatterv` makes sure that we scatter the entire image regardless of the size. We will define a local chunk of image as `local_x`.
```
##DO THE REST OF THE CODE IN EVERY NODE##
t4 = MPI.Wtime()	# START THE TIMER TO FIND THE TIME REQUIRED TO RUN THE CODE
Inarray = comm.bcast(Inarray, root = 0)		# BROADCAST LOCAL IMAGE DIMENSIONS TO EVERY NODE
#no_rows = comm.bcast(no_rows, root = 0)	# BROADCAST LOCAL IMAGE DIMENSIONS TO EVERY NODE
#local_c = comm.bcast(local_c, root = 0)
no_rows = Inarray[0]
local_c = Inarray[1]
local_x = np.empty((no_rows, local_c),dtype='uint8')	# DEFINE LOCAL IMAGE ON EVERY NODE
comm.Scatterv(img,local_x,root = 0)	# SCATTER THE IMAGE TO EVERY NODE
```
Now that the image is in respective nodes, we can count the number of stars in each of them. We will first define two variables `local_star_count` and `total_star_count` to hold the local and total star counts.
```
local_star_count = np.arange(1)		# DEFINE THE LOCAL AND TOTAL STAR COUNT VARIABLES
total_star_count = np.arange(1)
```
Step 5: A star in the image is found out by using the adaptiveThreshold method in OpenCV. The following lines of code does the same.
```
img_node_thresh =  cv2.adaptiveThreshold(local_x,255,cv2.ADAPTIVE_THRESH_MEAN_C,cv2.THRESH_BINARY,59,0) # LOGIC TO COUNT STARS
local_star_count = ((200 < img_node_thresh)).sum()	# LOGIC TO COUNT STARS IN EACH NODE
```
Step 6: Now we have the star-count in each of the nodes. We will go ahead and print these out along with the rank of the node. Then, the total star count is found out using the `Reduce` function and using `MPI.SUM` operation to the master node.
```
print "Number of stars in Node", rank, "is: ", local_star_count	# PRINT THE VALUE OF STAR COUNT IN EACH NODE
comm.Reduce(local_star_count, total_star_count, op = MPI.SUM, root = 0)	# REDUCE THE LOCAL STAR COUNTS INTO TOTAL STAR COUNT VARIABLE USING REDUCE FUNCTION
```
Step 7: Finally, in the master node, the timer is stopped, the time is logged and displayed along with the total star-count. The image is displayed, if needed.
```
if rank == 0:			# IN THE MASTER NODE
	t5 = MPI.Wtime()	# STOP THE TIMER
	print "Time required to complete the code is: ", t5-t4 	# PRINT THE TIME REQUIRED TO RUN THE CODE
	print "Total star count is: ", total_star_count		# PRINT THE TOTAL NUMBER OF STARS
	#cv2.imshow("galaxy.jpg",img)
	#cv2.waitKey(0)
```

