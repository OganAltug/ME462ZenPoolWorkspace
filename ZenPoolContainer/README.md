# ROS2_Humble_Container
basic humble container template

example run command:
docker run -it \ 
--user ros \ 
--network=host --ipc=host \ 
-v $PWD:/home/ros/ros2_ws \ 
-v /tmp/.X11-unix:/tmp/X11-unix.rw --env=DISPLAY \ 
-v /dev:/dev --device-cgroup-rule="c *:* rmw" \ 
humble_osrf 

in sequential order, parameter meanings:
#run interactive container with terminal
#specify user
#allow network access ?
#mount workspace into container
#setup to use gui apps within container
#allow pheripheral device access
#image name
