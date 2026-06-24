# Opera-Sim hackathon Sample Code


https://github.com/user-attachments/assets/c312b7c1-5d2b-4232-a7d8-8e70a7bcc8b6


## 依存パッケージ
- 以下のパッケージをros2_ws/srcにcloneする.
- [ros_tcp_endpoint](https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git)
- [ic120_ros2](https://github.com/pwri-opera/ic120_ros.git)
- [task_manager](https://github.com/Ryoya1012/Opera-sim_hackathon-Sample.git)
- [zx200_autonomy](https://github.com/Ryoya1012/ZX200_autonomy_state_machine.git)
- [ic120_autonomy](https://github.com/Ryoya1012/Opera-sim_ic120_hackathon.git)

```bash
# example
$ cd ~/ros2_ws/src
$ git colne https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git
```

## build手順
```bash
$ cd ~/ros2_ws
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
### Unity側での設定
- UnityEditorの上部ツールバーからRobotics->ROS Settingを開き, "ROS IP Adress", "ROS Port"のところにROS側のIPアドレスおよびポート番号(default : 10000)を入力.
- "Protocol"が"ROS2"になっていることを確認.

<img width="850" height="1377" alt="ROS-TCP-Connector" src="https://github.com/user-attachments/assets/5f94d2f4-8980-4eb5-bc0a-0676636a3a0c" />


<img width="2313" height="535" alt="Connection_Statue" src="https://github.com/user-attachments/assets/e7a2ce7c-30a3-4f80-b7ed-09e40f96d42e" />


- 正常に接続されると, LinuxPCのターミナルにOKとlogが表示される.
  
`[default_server_endpoint-1] [INFO] [1782343553.791738689] [UnityEndpoint]: RegisterPublisher(clock, <class 'rosgraph_msgs.msg._clock.Clock'>) OK`


2. Linux PCとシミュレータPC間の通信ができたら, Linux PCで別のターミナルを立ち上げ以下を実行する.
```bash
$ ros2 launch task_manager hackathon_launch.py
```

すると, zx200(ドラグショベル)が指定回数の掘削・積込動作を行い, ic120(クローラダンプ)が移動.
指定された放土位置に到着後, ベゼル(荷台)を上げ下げし放土を行う. 放土完了後, ic120は, はじめの位置に移動し, サンプルコードが終了する.

