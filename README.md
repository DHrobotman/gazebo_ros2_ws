# gazebo_ros2_control


作者在需要在ROS2 humble Gazebo11环境中仿真两个robot，区分为robot1和robot2，单独一个机器人在启动时不会发生任何异常，日志及效果均正常，同时整个控制通路也是正常，这说明我们在单个机器人上的namespace加载是正确的。但是通过区分namespace启动两个机器人的时候，ros2 node list 会出现以下错:


当先启动robot1的时候，我们同时将namespace传入了xacro文件当中，会发现：

/robot1/robot1/imu_plugin
/robot1/robot2/imu_plugin


而当先启动robot2，则变为：

/robot2/robot1/imu_plugin
/robot2/robot2/imu_plugin

问题出在gazebo_ros2_control这个包， 需要修改重新编译，
本仓库提供了关于ubuntu22.04 humble版本 gazebo_ros2_control功能包修改后的版本
具体问题和步骤查看https://blog.csdn.net/Dhrobotman/article/details/151968658?spm=1001.2014.3001.5502
