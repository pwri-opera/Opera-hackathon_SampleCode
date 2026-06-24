# Opera-Sim ハッカソンサンプルコード実行手順
- ここで使用するハッカソンサンプルコードは, ROS2 Humble環境で動作確認を行っているため, 事前にROS2のバージョンを確認しておくこと.

- 確認方法
```bash
$ echo $ROS_DISTRO
```

## 依存パッケージ
- 以下のパッケージをros2_ws/srcにcloneする.
- [ros_tcp_endpoint](https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git)
- [ic120_ros2](https://github.com/pwri-opera/ic120_ros.git)
- [task_manager](https://github.com/Ryoya1012/Opera-sim_hackathon-Sample.git)
- [zx200_autonomy](https://github.com/Ryoya1012/ZX200_autonomy_state_machine.git)
- [ic120_autonomy](https://github.com/Ryoya1012/Opera-sim_ic120_hackathon.git)

```bash
# example
$ git colne https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git
```

## build手順
```bash
$ cd ~/ros2_ws/src
```
各パッケージのブランチを確認.
本サンプルコードを実行するためには以下の条件でビルドを行う.

|package|Branch to specify|
|:---|:---|
|ros_tcp_endpoint|main|
|ic120_ros|main|
|task_manager|main|
|zx200_autonomy|hackathon|
|ic120_autonomy|main|

- 各パッケージが指定したブランチにいることを確認した後, 以下の通りにbuildを行う.
```bash
$ colcon build --packages-select ic120_navigation task_manager zx200_autonomy ic120_autonomy ros_tcp_endpoint
```

- ビルド完了後, 以下の通りにsouceを行う.
※ これを行わないと, buildを行った結果がrosシステムに反映されない.

```bash
$ source install/setup.bash
```

## 実行方法(2つのターミナルを使用)

1. ROS2(Linux PC)とUnity(DAIV)をLANケーブルで接続する

```bash
$ ros2 launch ros_tcp_endpoint endpoint.py
#注意：endpoint.pyはtab補完が効かない為手入力を行う
```
- 正常に接続されると, LinuxPCのターミナルにOKとlogが表示される.

2. Linux PCとシミュレータPC間の通信ができたら, Linux PCで別のターミナルを立ち上げ以下を実行する.
```bash
$ ros2 launch task_manager hackathon_launch.py
```

すると, zx200(ドラグショベル)が指定回数の掘削・積込動作を行い, ic120(クローラダンプ)が移動.
指定された放土位置に到着後, ベゼル(荷台)を上げ下げし放土を行う. 放土完了後, ic120は, はじめの位置に移動し, サンプルコードが終了する.

