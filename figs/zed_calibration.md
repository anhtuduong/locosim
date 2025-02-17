

# Zed camera calibration

Inside the zed_wrapper package, I implemented a  calibration procedure  for the ZED2  camera using an Aruco marker. In essence you need to run the script:

```
rosrun zed_wrapper camera_calibration.py
```

this will run two nodes: 1) the zed_wrapper driver, 2) the aruco_ros package. Then the script will get the TF between the camera and the Aruco marker, and, since  the position of the Aruco marker w.r.t. the Base Frame is suposed to be known  (I measured it),  by the law of composition of transforms it spits out on screen the TF  (3D position of the origin of the frame + 3 values for the Euler Angles for the orientation) between the Base Frame and the camera frame. Every time yo do a new calibration, you should set the TF obtained from this  procedure, in here:

https://github.com/mfocchi/ur_description/blob/5877efc9b53bf04912d2b8d29850eb70f005d26e/sensors/zed2/real/zed2.launch#L33

You can check the values of the point cloud, uncommenting this line

https://github.com/mfocchi/robot_control/blob/10bc564604a3337b2cc38cc8555d0cadefccc7e4/lab_exercises/lab_palopoli/ur5_generic.py#L268

note that the TF obtained in the calibration is wr.t. the base frame, so, if you want the data in the World Frame you should consider these offset and rotation for the mapping:

https://github.com/mfocchi/robot_control/blob/10bc564604a3337b2cc38cc8555d0cadefccc7e4/lab_exercises/lab_palopoli/ur5_generic.py#L184

https://github.com/mfocchi/robot_control/blob/10bc564604a3337b2cc38cc8555d0cadefccc7e4/lab_exercises/lab_palopoli/ur5_generic.py#L185



# Zed camera usage

The resolution set by default both in the real robot in simulation is (1080p = 1920x1080), you can check in simulation the resolution in this commit:

https://github.com/mfocchi/ur_description/commit/2c45a3294876a55c4c5b0439266f9c62a52bdf33

To get the 3D point data you need to subscribe to the "/ur5/zed_node/point_cloud/cloud_registered" message:

https://github.com/mfocchi/robot_control/blob/traj_optimization/lab_exercises/lab_palopoli/ur5_generic.py#L140

The  point-cloud is published in different frames, in simulation and on the real robot. On the real robot is "zed2_camera_center", while in simulation is  "zed2_left_camera_optical_frame". Therefore you need to consider  different transforms in the two cases to have the point cloud data  expressed in the world file, as I do here (since they are fixed  transforms you can print and copy them in your code):

https://github.com/mfocchi/robot_control/blob/traj_optimization/lab_exercises/lab_palopoli/ur5_generic.py#L177

To extract the data from the PointCloud2 message you need to provide the  points in the image plane for  which you want to know the 3D coordinates. To do this you should call the function read_points passing the indexes (u,v) as done here:

https://github.com/mfocchi/robot_control/blob/traj_optimization/lab_exercises/lab_palopoli/ur5_generic.py#L264

Unfortunately, these indexes (u,v) do not range across the full resolution but only  the half of it (i.e. u= [0..960], v=[0..540]), so to get the center of  the image you need to call with u =480 v =270. The center of the image  will correspond to the origin of the camera frame, which, as I said, is  different on real robot and simulation. I summarized everything in this  drawing:

https://github.com/mfocchi/locosim/blob/develop/figs/zed_camera.pdf

If you are using Yolo for object detection this library returns points  belonging to a squared image 640x640 (whatever resolution you give in  input), so you need to multiply the value of indexes that you get in YOLO by different scaling factors: 

s_u = 960/640 (for u)

s_v = 540/640 (for v)

once you get the 3D point, to get meaningful results for your planning, you should map it to the world frame considering the appropriate rotation  matrix w_R_c and offset w_x_c of the camera frame origin that you can obtain here:

https://github.com/mfocchi/robot_control/blob/10bc564604a3337b2cc38cc8555d0cadefccc7e4/lab_exercises/lab_palopoli/ur5_generic.py#L184

https://github.com/mfocchi/robot_control/blob/10bc564604a3337b2cc38cc8555d0cadefccc7e4/lab_exercises/lab_palopoli/ur5_generic.py#L185

