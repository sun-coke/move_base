# move_base导航栈调用A*算法作为全局路径规划
## 1、通过git下载导航栈源文件并在工作空间下编译

```
cd catkin_ws/src
git clone https://github.com/ros-planning/navigation.git
cd catkin_ws && catkin_make
```

## 2、修改move_base.cpp文件中的全局规划器调用参数和动态参数文件
由默认的navfn包插件更改为global_planner包插件，插件名由blp_plugin.xml定义。

```
private_nh.param("base_global_planner", global_planner, std::string("global_planner/GlobalPlanner"));
```
同时将MoveBase.cfg中的参数作对应更改。

## 3、修改global_planner包中的planner_core.cpp
将"use_dijkstra"参数设置成false，调用A*算法。
```
private_nh.param("use_dijkstra", use_dijkstra, false);
```

## 4、修改A*算法成本参数值
在global_planner包中对GlobalPlanner.cfg文件中的成本值参数进行修改，解决规划路径出现贴近障碍物的情况。
```
gen.add("lethal_cost", int_t, 0, "Lethal Cost", 230, 1, 255)
gen.add("neutral_cost", int_t, 0, "Neutral Cost", 30, 1, 255)
gen.add("cost_factor", double_t, 0, "Factor to multiply each cost from costmap by", 3.0, 0.01, 5.0)
gen.add("publish_potential", bool_t, 0, "Publish Potential Costmap", True)
```

## 5、运行
此时已经将全局规划器由默认的Dijkstra算法改成A*算法，运行仿真程序进行检验。（注：代码修改后需要重新编译生效）
```
$ roscore
$ roslaunch turtlebot3_gazebo turtlebot3_world.launch
$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
$ rqt_graph
```
由通信逻辑图可以观察到全局规划器已经修改成了move_base/GlobalPlanner/plan，并利用A*算法进行全局规划。
