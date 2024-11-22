---
layout:     post
title:      MATLAB与ROS相关部分源码解读
subtitle:   本科毕业设计内容探索过程记录
date:       2024-11-19
author:     Space
header-img: img/the-first.png
catalog:   true
tags:
    - 本科毕设探索

---





# MATLAB与ROS相关部分源码解读

## cs.m

这段代码的主要功能是通过 MATLAB 控制 ROS 系统下的机器人进行运动。它首先连接 ROS 网络，然后创建并配置一个发布器，接着通过发送包含线性和角速度信息的消息来控制机器人前进并旋转，最后通过发送停止指令使机器人停止。

注意机器人的ip地址正确，且在机器人端运行`roscore`和`roslaunch base_control base_control.launch`指令，才可以建立连接，否则会运行失败报错。而且注意话题名字是否正确。

### 1. 断开连接
```matlab
rosshutdown
```
- **作用**：`rosshutdown` 是用来关闭当前的 ROS 会话，断开 MATLAB 与 ROS 系统的连接。如果你之前已经通过 `rosinit` 启动了一个 ROS 会话，执行 `rosshutdown` 会终止当前的 ROS 会话。

### 2. 连接到 ROS 主机
```matlab
rosinit('192.168.38.85')
```
- **作用**：`rosinit` 用来初始化一个与 ROS 网络的连接，指定 ROS 主机的 IP 地址。在这里，`'192.168.38.85'` 是连接的 ROS 主机的 IP 地址。执行这个命令后，MATLAB 会与 ROS 网络进行连接，允许 MATLAB 通过 ROS 系统进行机器人控制和数据交互。

### 3. 列出 ROS 节点与话题
```matlab
rosnode list
rostopic list
```
- **作用**：
  - `rosnode list`：列出当前 ROS 网络中所有的节点（node）。节点是 ROS 系统中的基本组成部分，负责执行实际的操作。
  - `rostopic list`：列出当前 ROS 网络中所有的主题（topic）。主题是用于节点间通信的媒介，可以用来发布或订阅消息。

执行这两个命令时，你可以查看到当前 ROS 系统上所有的节点和话题，用于确认你要与之交互的节点和话题是否已经存在。

### 4. 发布速度指令控制机器人运动 
```matlab
velcmd = rospublisher("/robot/cmd_vel")
```
- **作用**：`rospublisher` 用来创建一个发布器（publisher），该发布器负责向 ROS 网络中的指定主题（topic）发布消息。在这个例子中，`/robot/cmd_vel` 是机器人控制运动的主题，`cmd_vel` 通常用于发送速度命令。
- **解释**：你使用这个命令创建了一个发布器，准备向 `/robot/cmd_vel` 这个主题发布速度信息。

### 5. 创建并配置 ROS 消息
```matlab
vel = rosmessage(velcmd)
```
- **作用**：`rosmessage` 用来创建一个 ROS 消息对象，`velcmd` 是我们刚刚创建的发布器。`rosmessage` 函数会返回一个消息对象，该对象具有与发布器关联的数据结构和字段。
- **解释**：`vel` 是一个消息对象，用来存储机器人速度的相关数据。

### 6. 设置机器人线性和角速度
```matlab
vel.Linear.X = 0.2;
vel.Angular.Z = -0.5;
```
- **作用**：
  - `vel.Linear.X = 0.2;` 设置机器人的前进速度为 0.2 m/s（线性速度），即机器人会以 0.2 m/s 的速度沿着 X 轴方向（通常是前进方向）移动。
  - `vel.Angular.Z = -0.5;` 设置机器人的角速度为 -0.5 rad/s（角速度），即机器人将围绕 Z 轴（通常是垂直轴）旋转，负值表示逆时针旋转。

这两行代码控制了机器人同时进行平移和旋转，向前以 0.2 m/s 的速度运动，并且以 -0.5 rad/s 的速度进行旋转。

### 7. 发送运动指令
```matlab
send(velcmd, vel)
```
- **作用**：`send` 函数用于将消息 `vel` 通过发布器 `velcmd` 发送到 ROS 系统。也就是说，这行代码将控制机器人的运动，指示它按照设定的线性和角速度运动。

### 8. 停止运动 
```matlab
vel.Linear.X = 0.0;
vel.Angular.Z = 0.0;
```
- **作用**：这两行代码将机器人运动的线性速度和角速度都设置为 0，表示机器人停止运动。此时，机器人将不再向前移动，也不再旋转。

### 9. 发送停止指令
```matlab
send(velcmd, vel)
```
- **作用**：再次通过发布器 `velcmd` 将停止运动的消息 `vel` 发送到 ROS 网络，从而让机器人停止运动。

---



## ditu.m

这段MATLAB代码主要用于创建一个机器人仿真环境，其中包括地面、外墙和内墙的构建，以及生成二进制占据地图（`binaryOccupancyMap`）用于路径规划。

### 1. 创建机器人场景
```matlab
scenario = robotScenario(UpdateRate=5);
```
- **作用**：`robotScenario` 用于创建一个机器人仿真场景。该场景用于模拟机器人操作的环境。
  - `UpdateRate=5` 表示场景更新的频率是每秒更新 5 次。

### 2. 设置地面颜色并添加到场景
```matlab
floorColor = [0.5882 0.2941 0];
addMesh(scenario,"Plane",Position=[3 1.5 0],Size=[6 3],Color=floorColor);
```
- **作用**：
  - `floorColor` 定义了地面的颜色（RGB 值），这里是一个棕色的颜色。
  - `addMesh` 用于在场景中添加一个网格模型（mesh），这里添加的是一个平面（`Plane`）。
  - `Position=[3 1.5 0]` 指定了平面的位置，在 `(3, 1.5, 0)` 处。
  - `Size=[6 3]` 指定平面的大小是 6x3 的矩形。
  - `Color=floorColor` 设置平面的颜色为棕色。

这行代码实际上是把一个颜色为棕色的地面添加到场景的 `(3, 1.5, 0)` 位置，大小为 6x3 的矩形。

### 3. 设置墙壁的尺寸和颜色
```matlab
wallHeight = 0.4;
wallWidth = 0.06;
wallLength = 6;
wallColor = [1 1 0.8157];
```
- **作用**：
  - `wallHeight` 设置墙壁的高度为 0.4。
  - `wallWidth` 设置墙壁的宽度为 0.06。
  - `wallLength` 设置墙壁的长度为 6。
  - `wallColor` 设置墙壁的颜色（RGB 值），这里是一个淡黄色。

### 4. 添加外墙
这部分代码通过 `addMesh` 将外墙添加到场景中：

#### 下墙（位于底部）
```matlab
addMesh(scenario,"Box",Position=[wallLength/2, wallWidth/2, wallHeight/2],...
    Size=[wallLength, wallWidth, wallHeight],Color=wallColor,IsBinaryOccupied=true);
```
- **作用**：在场景中添加一个长方体（`Box`），作为底墙。
  - `Position=[wallLength/2, wallWidth/2, wallHeight/2]` 表示长方体的中心位置，计算得出应该放在 `(3, 0.03, 0.2)` 位置（即在底面）。
  - `Size=[wallLength, wallWidth, wallHeight]` 设置底墙的长宽高分别为 6, 0.06, 0.4。
  - `Color=wallColor` 设置墙壁颜色为淡黄色。
  - `IsBinaryOccupied=true` 表示这个墙壁是占据空间的，可以被视为一个“障碍物”在地图上。

#### 左墙
```matlab
addMesh(scenario,"Box",Position=[wallWidth/2, wallLength/4, wallHeight/2],...
    Size=[wallWidth, wallLength/2, wallHeight],Color=wallColor,IsBinaryOccupied=true);
```
- **作用**：在场景中添加左侧墙壁，位置、尺寸和颜色都类似于下墙。

#### 右墙
```matlab
addMesh(scenario,"Box",Position=[wallLength-wallWidth/2, wallLength/4, wallHeight/2],...
    Size=[wallWidth, wallLength/2, wallHeight],Color=wallColor,IsBinaryOccupied=true);
```
- **作用**：在场景中添加右侧墙壁，位置、尺寸和颜色与左墙相同。

#### 上墙
```matlab
addMesh(scenario,"Box",Position=[wallLength/2, wallLength/2-wallWidth/2, wallHeight/2],...
    Size=[wallLength, wallWidth, wallHeight],Color=wallColor,IsBinaryOccupied=true);
```
- **作用**：在场景中添加上墙，位置、尺寸和颜色与下墙相同。

### 5. 添加内墙
```matlab
addMesh(scenario,"Box",Position=[wallLength-1.5, wallLength/2-0.5, wallHeight/2],...
    Size=[wallWidth, 1, wallHeight],Color=wallColor,IsBinaryOccupied=true);
```
- **作用**：在场景中添加一个内部墙壁，位置为 `(wallLength-1.5, wallLength/2-0.5, wallHeight/2)`，它的宽度为 0.06（与外墙相同），长度为 1，高度为 0.4。这个墙壁是用于将空间分割成不同的区域。

### 6. 显示场景和设置视角
```matlab
show3D(scenario);
lightangle(-45,30);
view(60,50);
```
- **作用**：
  - `show3D(scenario)`：显示当前创建的三维场景。
  - `lightangle(-45,30)`：设置光源的角度，控制场景的光照效果。
  - `view(60,50)`：设置相机的视角，使得仿真场景的显示更加清晰。

### 7. 创建并保存二进制占据地图 
```matlab
map = binaryOccupancyMap(scenario, GridOriginInLocal=[-0.1 -0.1], MapSize=[6.1 3.1], MapHeightLimits=[0 3]);
save map.mat map
```
- **作用**：
  - `binaryOccupancyMap` 用于创建一个二进制占据地图，它会从仿真场景中生成一个表示环境占据状态的地图。占据地图是路径规划中的常用工具，其中 `0` 表示没有障碍物的区域，`1` 表示有障碍物的区域。
  - `GridOriginInLocal=[-0.1 -0.1]` 设置地图的局部原点相对于场景坐标系的偏移。
  - `MapSize=[6.1 3.1]` 设置地图的大小为 6.1 x 3.1。
  - `MapHeightLimits=[0 3]` 设置地图的高度范围为 0 到 3（通常用于三维地图的限制）。
  - `save map.mat map`：将生成的地图保存为文件 `map.mat`。

### 8. 显示生成的占据地图
```matlab
show(map)
```
- **作用**：显示刚刚创建的二进制占据地图。

### 9. 地图膨胀（注释掉的部分）
```matlab
%inflate(map,0.15);
```
- **作用**：这行代码被注释掉了，如果启用，`inflate(map,0.15)` 会对地图进行膨胀操作，增大障碍物的范围，通常用于路径规划中考虑到机器人尺寸时的安全空间。



## ligh.m

这段代码演示了如何与 ROS 机器人进行通信、控制其运动并实现轨迹跟踪。使用了两种方式来获取机器人的位姿，并使用 `Pure Pursuit` 控制器实现轨迹跟踪。整体流程涉及 ROS 节点初始化、话题订阅、消息发布、速度控制和轨迹跟踪控制等步骤。

### 1. 断开连接与初始化连接
```matlab
rosshutdown
rosinit('192.168.137.39')
```
- **`rosshutdown`**：关闭当前的 ROS 连接。
- **`rosinit('192.168.137.39')`**：初始化与机器人进行通信，连接到指定的 IP 地址（假设机器人运行在这个 IP 上）。

### 2. 查看 ROS 节点与话题
```matlab
rosnode list
rostopic list
```
- **`rosnode list`**：列出当前 ROS 系统中所有的节点。
- **`rostopic list`**：列出当前 ROS 系统中所有的主题。

### 3. 发布速度命令（控制机器人运动）
```matlab
velcmd = rospublisher("/robot0/cmd_vel");
vel = rosmessage(velcmd);
vel.Linear.X = 0.2;
vel.Angular.Z = -0.5;
send(velcmd, vel)
```
- **`rospublisher("/robot0/cmd_vel")`**：创建一个发布 `/robot0/cmd_vel` 话题的发布者，这个话题通常用来控制机器人的速度。
- **`rosmessage(velcmd)`**：创建一个与 `velcmd` 话题相关联的消息对象。
- **`vel.Linear.X = 0.2;`**：设置线速度为 0.2 米/秒。
- **`vel.Angular.Z = -0.5;`**：设置角速度为 -0.5 弧度/秒，表示机器人朝逆时针方向旋转。
- **`send(velcmd, vel)`**：将速度消息发送到 `/robot0/cmd_vel` 话题，从而控制机器人进行运动。

### 4. 停止机器人运动
```matlab
vel.Linear.X = 0.0;
vel.Angular.Z = 0.0;
send(velcmd, vel)
```
- 这段代码将机器人的线速度和角速度都设置为零，从而停止机器人。

### 5. 获取机器人位姿（两种方式）
#### 方式一：直接获取位姿
```matlab
Odometry = rostopic("echo", "/robot0/odom");
showdetails(Odometry);
newpos = Odometry.Pose.Pose.Position;
```
- **`rostopic("echo", "/robot0/odom")`**：订阅 `/robot0/odom` 话题，获取机器人的里程计数据（包括位姿、速度等信息）。
- **`showdetails(Odometry)`**：显示 `Odometry` 消息的详细信息。
- **`newpos = Odometry.Pose.Pose.Position;`**：提取机器人的位置（坐标）。

#### 方式二：通过订阅器获取位姿
```matlab
myloc = rossubscriber("/odom");
Odomdate = myloc.LatestMessage;
newpos = Odomdate.Pose.Pose.Position;
```
- **`rossubscriber("/odom")`**：创建一个订阅器，订阅 `/odom` 话题，获取机器人的位姿数据。
- **`myloc.LatestMessage`**：获取最新的位姿信息。
- **`newpos = Odomdate.Pose.Pose.Position;`**：提取当前位置坐标。

### 6. 获取新的位姿并计算机器人的当前位置
```matlab
Odometry = rostopic("echo", "/odom");
showdetails(Odometry);
newpos = Odometry.Pose.Pose.Position;
x0 = newpos.X + 10;
y0 = newpos.Y + 10;
Delta = Odometry.Pose.Pose.Orientation;
Y = Delta.Y;
X = Delta.X;
theta0 = Y / X;
robotCurrentPose = [x0, y0, theta0];
```
- 获取机器人的位姿信息，并计算其新的位姿（`x0, y0, theta0`），更新机器人的当前位置。

### 7. 轨迹跟踪控制（Pure Pursuit 控制器）
```matlab
controller = controllerPurePursuit;
controller.Waypoints = path;
controller.DesiredLinearVelocity = ...;
controller.DesiredAngularVelocity = ...;
```
- **`controllerPurePursuit`**：使用纯追踪（Pure Pursuit）控制器来进行轨迹跟踪。
- **`controller.Waypoints = path;`**：设置轨迹的目标点（waypoints），通常 `path` 是一个包含多个路径点的矩阵。
- **`controller.DesiredLinearVelocity`** 和 **`controller.DesiredAngularVelocity`**：设置机器人沿轨迹运动时的期望线速度和角速度。

### 8. 发布速度控制命令
```matlab
velcmd = rospublisher("/cmd_vel");
vel = rosmessage(velcmd);
vel.Linear.X = 0.2;
vel.Angular.Z = 0.5;
send(velcmd, vel);
```
- 发布新的速度控制命令，控制机器人沿轨迹运动。
- 这里将线速度设置为 0.2 米/秒，角速度设置为 0.5 弧度/秒。

### 9. 停止运动
```matlab
vel.Linear.X = 0.0;
vel.Angular.Z = 0.0;
send(velcmd, vel);
```
- 在轨迹跟踪完成后，停止机器人运动。



## map.mat

![img](https://raw.githubusercontent.com/dhw2536/Picture/main/202411191842137.png)

这个图像显示的是一个名为 `binaryOccupancyMap` 的对象的属性，它看起来像是一个二进制占用栅格地图（Binary Occupancy Map），通常用于机器人学和自动化领域来表示环境的占用情况。下面是各个属性的解释：

1. **LayerName**: 'binaryLayer' - 这可能是地图的名称或者标识。

2. **DataType**: 'logical' - 表示数据类型是逻辑型，通常用于表示真（1）或假（0）。

3. **DefaultValue**: 0 - 默认值，可能表示未占用或未知状态。

4. **DataSize**: [31,61] - 这表示地图的尺寸，有31行和61列。

5. **GetTransformFcn**: // - 这可能是一个函数句柄，用于获取地图的转换信息，但这里没有具体信息。

6. **SetTransformFcn**: // - 这可能是一个函数句柄，用于设置地图的转换信息，同样这里没有具体信息。

7. **GridLocationInWorld**: [-0.1000;-0.1000] - 这表示地图在世界坐标系中的位置。

8. **GridOriginInLocal**: [-0.1000;-0.1000] - 这表示地图在本地坐标系中的原点位置。

9. **LocalOriginInWorld**: [0,0] - 这表示本地坐标系在世界坐标系中的原点位置。

10. **Resolution**: 10 - 这可能表示地图的分辨率，即每个单元格在世界坐标系中的大小。

11. **GridSize**: [31,61] - 这与DataSize相同，再次确认地图的尺寸。

12. **XWorldLimits**: [-0.1000,6] - 这表示地图在X轴方向上的世界坐标范围。

13. **YWorldLimits**: [-0.1000,3] - 这表示地图在Y轴方向上的世界坐标范围。

14. **XWorldLimits**: [-0.1000,6] - 这个属性重复了，可能是一个错误。

这个对象可能是在MATLAB环境中使用的，用于机器人路径规划或环境建模。每个单元格的值可能表示该位置是否被占用（例如，1表示占用，0表示空闲）。



## PRM sw.m

这段 MATLAB 代码展示了如何使用 ROS 与机器人进行通信、控制其运动，并实现路径规划与轨迹跟踪。代码的流程从路径规划、控制器设置到与机器人的实时交互，涉及了多方面的技术细节。

### 1. **初始化和地图加载**
```matlab
clc; clear; close all;
sampleTime = 0.01; % 设置采样时间
% rosshutdown % 关闭 ROS 连接（如果之前已经连接）
```
- **`clc; clear; close all;`**：清空命令窗口，清除工作区变量，并关闭所有图形窗口。
- **`sampleTime = 0.01;`**：定义控制器的采样时间（用于周期性更新控制器的输出）。
- **`rosshutdown`**：如果之前已连接到 ROS 系统，此命令关闭 ROS 连接。这里是注释掉的，但可以在实际使用时启用。

```matlab
% load exampleMaps
% map = binaryOccupancyMap(simpleMap);
load map.mat
figure
show(map)
```
- **`load map.mat`**：加载一个已保存的地图 `map.mat`。这个地图可以是一个二值占用地图，表示环境中的障碍物与空旷区域。
- **`figure; show(map)`**：创建一个新的图形窗口并显示地图。

### 2. **地图膨胀**
```matlab
mapInflated = copy(map); % 复制地图
inflate(mapInflated, 0.4); % 对地图进行膨胀处理，膨胀0.4米
figure
show(mapInflated)
```
- **`mapInflated = copy(map);`**：复制原始地图。
- **`inflate(mapInflated, 0.4);`**：对地图进行膨胀，以确保机器人能够避开障碍物。膨胀半径为 0.4 米，这意味着机器人与障碍物之间至少要有 0.4 米的安全距离。
- **`figure; show(mapInflated)`**：显示膨胀后的地图。

### 3. **路径规划：使用 PRM (概率路线图)**
```matlab
prm = robotics.PRM(mapInflated); % 创建概率路线图 (PRM) 规划器
prm.NumNodes = 1000; % 设置 PRM 中的节点数量
prm.ConnectionDistance = 0.7; % 设置连接距离，即每两个节点之间可以连接的最大距离
```
- **`prm = robotics.PRM(mapInflated);`**：使用膨胀后的地图创建一个概率路线图（PRM）路径规划器。
- **`prm.NumNodes = 1000;`**：在 PRM 中生成 1000 个随机节点。
- **`prm.ConnectionDistance = 0.7;`**：设置节点之间可以连接的最大距离。距离太大可能导致路径规划不够精细。

```matlab
startLocation = [0.8 0.8]; % 设置起始位置
endLocation = [5 1.5]; % 设置目标位置
path = findpath(prm, startLocation, endLocation); % 计算从起始位置到目标位置的路径
```
- **`startLocation = [0.8, 0.8];`**：定义机器人起始位置（x, y）。
- **`endLocation = [5, 1.5];`**：定义机器人目标位置（x, y）。
- **`path = findpath(prm, startLocation, endLocation);`**：通过 PRM 规划器找到从起始位置到目标位置的路径。

```matlab
pathLength = 0;
for i = 2:length(path)
    pathLength = pathLength + norm(path(i,:) - path(i-1,:)); % 计算路径长度
end
disp(['路径长度为：', num2str(pathLength)]);
save path.mat path % 保存路径数据
show(prm) % 显示 PRM 的节点和路径
```
- 计算路径的总长度并输出。
- **`save path.mat path`**：将计算得到的路径保存到文件 `path.mat` 中，以便后续使用。
- **`show(prm)`**：在图形中显示 PRM 路线图及其生成的路径。

### 4. **设置纯追踪控制器 (Pure Pursuit Controller)**
```matlab
controller = controllerPurePursuit; % 创建纯追踪控制器
release(controller); % 释放之前的控制器设置
controller.Waypoints = path; % 设置路径点
controller.DesiredLinearVelocity = 0.1; % 设置线速度 0.1 米/秒
controller.MaxAngularVelocity = 0.2; % 设置最大角速度 0.2 弧度/秒
controller.LookaheadDistance = 0.8; % 设置前视距离 0.8 米
```
- **`controller = controllerPurePursuit;`**：创建一个纯追踪控制器对象。
- **`controller.Waypoints = path;`**：设置控制器的目标路径点为 `path`。
- **`controller.DesiredLinearVelocity = 0.1;`**：设置机器人沿路径的线速度为 0.1 米/秒。
- **`controller.MaxAngularVelocity = 0.2;`**：设置最大角速度为 0.2 弧度/秒，防止机器人转向过快。
- **`controller.LookaheadDistance = 0.8;`**：设置前视距离（即控制器所需的目标点到当前位置的最小距离），较大的值可以让机器人在路径上行驶更平稳，但可能导致较大角度的转向。

### 5. **连接 ROS 机器人**
```matlab
rosinit('192.168.38.113') % 连接到 ROS 机器人
velcmd = rospublisher("/robot/cmd_vel"); % 创建 cmd_vel 话题的发布者
vel = rosmessage(velcmd); % 创建一个对应的消息对象
```
- **`rosinit('192.168.38.113')`**：通过给定的 IP 地址初始化与 ROS 系统的连接。
- **`rospublisher("/robot/cmd_vel")`**：创建一个发布 `/robot/cmd_vel` 话题的发布者，通常用于控制机器人运动。
- **`rosmessage(velcmd)`**：创建一个 ROS 消息对象，准备发送速度命令。

### 6. **获取机器人的初始位姿**
```matlab
Odometry = rostopic("echo", "/robot/odom"); % 获取里程计数据
newpos = Odometry.Pose.Pose.Position; % 提取当前位置坐标
x0 = newpos.X + 0.8; y0 = newpos.Y + 0.8; % 修正偏移
Delta = Odometry.Pose.Pose.Orientation; % 提取机器人的方向
Y = Delta.Z; X = Delta.W; % 提取方向四元数
theta0 = Y / X; % 计算机器人朝向
robotInitialLocation = path(1,:); % 获取路径起点
robotGoal = path(end,:); % 获取路径终点
robotCurrentPose = [x0, y0, theta0]; % 当前位姿
```
- **`Odometry = rostopic("echo", "/robot/odom");`**：获取机器人的里程计信息（包括位置和姿态）。
- **`newpos = Odometry.Pose.Pose.Position;`**：提取机器人的当前位置。
- **`x0, y0, theta0`**：根据里程计数据初始化机器人当前位置和朝向。

### 7. **机器人路径跟踪**
```matlab
distanceToGoal = norm(robotInitialLocation - robotGoal); % 计算到目标的距离
goalRadius = 0.1; % 设定目标半径

while distanceToGoal > goalRadius % 当距离目标大于目标半径时
    [v, omega] = controller(robotCurrentPose); % 获取速度命令
    vel.Linear.X = v; % 设置线速度
    vel.Angular.Z = omega; % 设置角速度
    send(velcmd, vel); % 发送速度命令到机器人

    Odometry = rostopic("echo", "/robot/odom"); % 更新机器人位置
    newpos = Odometry.Pose.Pose.Position;
    x1 = newpos.X + 0.8; y1 = newpos.Y + 0.8;
    Delta = Odometry.Pose.Pose.Orientation;
    Y = Delta.Z; X = Delta.W;
    theta1 = Y / X;
    robotCurrentPose = [x1, y1, theta1]; % 更新当前位置

    robotLocation = [x1, y1]; % 当前位姿
    distanceToGoal = norm(robotLocation - robotGoal); % 更新到目标的距离
end
```
- **`while distanceToGoal > goalRadius`**：当机器人距离目标位置大于设定的目标半径（0.1 米）时，控制器持续输出速度命令。

- **`[v, omega] = controller(robotCurrentPose);`**：调用纯追踪控制器，根据当前的机器人位姿（位置和朝向）计算出线速度 `v` 和角速度 `omega`。

  - `v` 是机器人沿着路径的线速度（单位：米/秒）。
  - `omega` 是机器人绕其垂直轴的角速度（单位：弧度/秒）。

- **`vel.Linear.X = v;`**：设置消息对象 `vel` 中的线速度为 `v`。

- **`vel.Angular.Z = omega;`**：设置消息对象 `vel` 中的角速度为 `omega`。

- **`send(velcmd, vel);`**：将速度命令通过 ROS 发布到 `/robot/cmd_vel` 话题上，这样机器人就会根据这些速度指令来运动。

  ### 8. **更新机器人位置和检测是否到达目标**
  ```matlab
  Odometry = rostopic("echo", "/robot/odom"); % 更新机器人位置
  newpos = Odometry.Pose.Pose.Position;
  x1 = newpos.X + 0.8; y1 = newpos.Y + 0.8;
  Delta = Odometry.Pose.Pose.Orientation;
  Y = Delta.Z; X = Delta.W;
  theta1 = Y / X;
  robotCurrentPose = [x1, y1, theta1]; % 更新当前位置
  ```
  - **`Odometry = rostopic("echo", "/robot/odom");`**：再次从 ROS 获取里程计数据，更新机器人的当前状态（位置和姿态）。
    
  - **`newpos = Odometry.Pose.Pose.Position;`**：提取最新的机器人位置信息。

  - **`x1 = newpos.X + 0.8; y1 = newpos.Y + 0.8;`**：机器人位置信息可能有一个初始偏移量，因此进行位置修正，确保与实际坐标一致。

  - **`Delta = Odometry.Pose.Pose.Orientation;`**：提取机器人的四元数姿态信息。

  - **`theta1 = Y / X;`**：根据四元数计算出机器人的朝向角度 `theta1`。

  - **`robotCurrentPose = [x1, y1, theta1];`**：更新机器人的当前位姿（位置和朝向）。

  ### 9. **计算到目标的距离**
  ```matlab
  robotLocation = [x1, y1]; % 当前位姿
  distanceToGoal = norm(robotLocation - robotGoal); % 更新到目标的距离
  ```
  - **`robotLocation = [x1, y1];`**：将机器人的当前位置信息保存在 `robotLocation` 中。

  - **`distanceToGoal = norm(robotLocation - robotGoal);`**：计算机器人当前位置与目标位置之间的距离。如果该距离小于目标半径 `goalRadius`，则表示机器人已到达目标。

  ### 10. **完成任务，停止机器人**
  ```matlab
  vel.Linear.X = 0.0; % 停止线速度
  vel.Angular.Z = 0.0; % 停止角速度
  send(velcmd, vel); % 发送停止命令到机器人
  rosshutdown; % 关闭 ROS 连接
  ```
  - **`vel.Linear.X = 0.0; vel.Angular.Z = 0.0;`**：当机器人到达目标位置时，停止发送速度命令，将线速度和角速度都设置为 0。

  - **`send(velcmd, vel);`**：将停止命令通过 ROS 发布到 `/robot/cmd_vel` 话题上，机器人会停止运动。

  - **`rosshutdown;`**：关闭与 ROS 的连接，断开通信。

  ### 总结
  这段代码的流程是典型的移动机器人路径规划和轨迹跟踪控制的实现。具体步骤如下：
  1. 加载和处理地图。
  2. 使用概率路线图（PRM）进行路径规划，找到从起点到终点的路径。
  3. 使用纯追踪控制器（Pure Pursuit Controller）计算出机器人需要的线速度和角速度。
  4. 在 ROS 环境下与机器人进行通信，通过发布控制指令来驱动机器人沿规划路径前进。
  5. 在机器人到达目标位置后，停止机器人并关闭 ROS 连接。

  这个过程主要涉及到路径规划、轨迹跟踪控制和与 ROS 机器人的交互，是实现自动化移动的关键步骤。

