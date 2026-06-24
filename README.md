# Opera-Sim ハッカソンサンプルコード実行手順

## 依存パッケージ
- 以下のパッケージをros2_ws/srcにcloneする.
- [ros_tcp_endpoint](https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git)
- [ic120_ros2](https://github.com/pwri-opera/ic120_ros.git)
- [task_manager](https://github.com/Ryoya1012/Opera-sim_hackathon-Sample.git)
- [zx200_autonomy](https://github.com/Ryoya1012/ZX200_autonomy_state_machine.git)
- [ic120_autonomy](https://github.com/Ryoya1012/Opera-sim_ic120_hackathon.git)


## build手順
```bash
$ cd ~/ros2_ws/src
```
各パッケージのブランチを確認.
本サンプルコードを実行するためには以下の条件でビルドを行う.

|package|指定branch|
|:---|:---|
|ros_tcp_endpoint|main|
|ic120_ros|main|
|task_manager|main|
|zx200_autonomy|hackathon|
|ic12_autonomy|main|

```bash
$ colcon build --packages-select ic120_navigation task_manager zx200_autonomy ic120_autonomy ros_tcp_endpoint
```

```bash
$ source install/setup.bash
```

## 実行方法(2つのターミナルを使用)
```bash
$ ros2 launch ros_tcp_endpoint endpoint.py
#注意：endpoint.pyはtab補完が効かない為手入力を行う
```

```bash
$ ros2 launch task_manager hackathon_launch.py
```

