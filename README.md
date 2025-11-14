# gazebo_ros2_control


作者在需要在ROS2 humble Gazebo11环境中仿真两个robot，区分为robot1和robot2，单独一个机器人在启动时不会发生任何异常，日志及效果均正常，同时整个控制通路也是正常，这说明我们在单个机器人上的namespace加载是正确的。但是通过区分namespace启动两个机器人的时候，ros2 node list 会出现以下错误：


当先启动robot1的时候，我们同时将namespace传入了xacro文件当中，会发现：

```
/robot1/robot1/imu_plugin
/robot1/robot2/imu_plugin
```

而当先启动robot2，则变为：

```
/robot2/robot1/imu_plugin
/robot2/robot2/imu_plugin
```

这里可以发现，先加载的namespace似乎被设置为全局变量后不再修改了，从而导致后加载的机器人模型被挂载在先加载的namespace下面，这显然是不对的。

通过广泛搜索，发现大家遇到的问题是一样的，问题出在`gazebo_ros2_control`这个包，将其加载注释后，其他所有配件的挂载namespace变的正常。有其他研究者分析，是因为在源码中，老版本的`gazebo_ros2_control`包中，`gazebo_ros2_control_plugin.cpp`这个文件，将namespace写成了全局参数，加载一次后就不会改变了，这导致了环境污染

```
// Set namespace if tag is present
if (sdf->HasElement("namespace")) {
  std::string ns = sdf->GetElement("namespace")->Get<std::string>();
  // prevent exception: namespace must be absolute, it must lead with a '/'
  if (ns.empty() || ns[0] != '/') {
    ns = '/' + ns;
  }
  std::string ns_arg = std::string("__ns:=") + ns;
  arguments.push_back(RCL_REMAP_FLAG);
  arguments.push_back(ns_arg);
}

 arguments.push_back(RCL_ROS_ARGS_FLAG);
```

一种解决方案是找到`gazebo_ros2_control_plugin.cpp`中以上代码，注释，重新编译，并按照下面的方法编译进`/opt/ros/humble`，即可解决这个问题，但是作者注释之后，发现编译无法通过。于是查看humble分支下面的`gazebo_ros2_control_plugin.cpp`，该分支已经解决了这个问题，因此拉下来切换到分支后，直接编译，并映射到`opt/ros/humble`即可解决。
解决方案，卸载`gazebo-ros2-control`，然后编译源码，并将源码映射到`/opt/ros/humble`

```
sudo apt remove ros-humble-gazebo-ros2-control
mkdir -p gazebo_ws/src
cd src
git clone https://github.com/ros-controls/gazebo_ros2_control.git
cd gazebo_ros2_control
git checkout humble

cd ~/gazebo_ws
colcon build
cd install/gazebo_ros2_control #进入要安装的功能包的目录，gazebo_ros2_control
# 安装进系统
sudo rsync -r  ./ /opt/ros/humble # 若将功能包安装到其他电脑系统，只需将install内的包拷贝到另一台电脑后进入包目录再行该命令即可
```

参考：
[https://fishros.org.cn/forum/topic/2777/ros2-humble%E4%B8%8B%E4%BD%BF%E7%94%A8gazebo%E4%BB%BF%E7%9C%9F%E5%A4%9A%E4%B8%AA%E6%9C%BA%E5%99%A8%E4%BA%BA](https://fishros.org.cn/forum/topic/2777/ros2-humble%E4%B8%8B%E4%BD%BF%E7%94%A8gazebo%E4%BB%BF%E7%9C%9F%E5%A4%9A%E4%B8%AA%E6%9C%BA%E5%99%A8%E4%BA%BA)
[https://github.com/ros-controls/gazebo_ros2_control/pull/181](https://github.com/ros-controls/gazebo_ros2_control/pull/181)
[https://github.com/ros-controls/gazebo_ros2_control/issues/127](https://github.com/ros-controls/gazebo_ros2_control/issues/127)
[https://github.com/ros-simulation/gazebo_ros_pkgs/issues/1392](https://github.com/ros-simulation/gazebo_ros_pkgs/issues/1392)

