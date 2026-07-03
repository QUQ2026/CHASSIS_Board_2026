底盘板（考核版）

硬件连线上：

can1接云台板（双板通信）和yaw轴电机 [未来还有拨弹和超级电容]
can2接四个底盘3508电机

逻辑上底盘板要干的事情：

1.接收云台板发来的信息，两部分

第一部分   0x231 遥控

uint8_t ChassisRXResolve(uint8_t *data,DBUS_Typedef *DBUS,RUI_ROOT_STATUS_Typedef *Root)

摇杆量还原到本地DBUS结构体，同时刷新RM_DBUS在线状态和电容使能标志

第二部分   0x233 IMU反馈的总角度

uint8_t ChassisRXResolve_Yaw(uint8_t *data, CONTAL_Typedef *CONTAL)中的

CONTAL->CG.YawTotalAngle_from_gimbal= YawFrame.yaw_abs;

经过角度归一化 来进行坐标系转化（作为坐标变换用的云台绝对朝向），结合传来的遥控信息

算出底盘vx,vy,并结合vw进行运动学逆解

算出每个轮子的目标RPM，进行PID速度环并输出（要限制功率）

2.底盘模式切换方面，RobotTask(1) 里的

（一开始的想法是用s2切换普通，小陀螺和底盘跟随，但是实际遥控车后和想象中有点出路，故修改调整以下切换模式）

Chassis_Auto_changeMode_DBUS 根据 DBUS.Remote.Dial 的值自动切换：

拨轮绝对值大于50时进入小陀螺模式（Chassis_Gyroscope_DBUS）

拨轮归中时进入底盘跟随云台模式（Chassis_follow_Gimbal）。

3.Yaw编码器读取方面，CAN1上直接接收Yaw轴反馈帧，本地算出 RELATIVE_ANGLE（云台相对底盘偏角），供底盘跟随模式的PID使用。



