# Opera-Sim hackathon Sample Code

https://github.com/user-attachments/assets/c312b7c1-5d2b-4232-a7d8-8e70a7bcc8b6


## 依存パッケージ
- 以下のパッケージをros2_ws/srcにcloneする.
- [ros_tcp_endpoint](https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git)
- [ic120_ros2](https://github.com/pwri-opera/ic120_ros2.git)
- [Opera-sim_hackathon-Sample](https://github.com/Ryoya1012/Opera-sim_hackathon-Sample.git)
- [ZX200_autonomy_state_machine](https://github.com/Ryoya1012/ZX200_autonomy_state_machine.git)
- [ic120_autonomy](https://github.com/Ryoya1012/Opera-sim_ic120_hackathon.git)
- [com3_ros](https://github.com/pwri-opera/com3_ros.git)

```bash
# example
cd ~/ros2_ws/src
git clone https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git
```

## build手順
```bash
cd ~/ros2_ws
```
各パッケージのブランチを確認.
本サンプルコードを実行するためには以下の条件でビルドを行う.

また, 本リポジトリにある<mark>""waypoint.csv"をros2_ws直下に保存</mark>しておく.

|package|Branch to specify|
|:---|:---|
|ros_tcp_endpoint|main-ros2|
|ic120_ros2|feature/tmp_ic120_unity_setting|
|Opera-sim_hackathon-Sample|main|
|ZX200_Autonomy_state_machine|hackathon|
|ic120_autonomy|main|
|com3_ros|main|

- 各パッケージが指定したブランチにいることを確認した後, 以下の通りにbuildを行う.
```bash
colcon build --symlink-install --packages-up-to ic120_navigation task_manager zx200_autonomy ic120_autonomy ic120_unity com3_msgs ros_tcp_endpoint
```

- ビルド完了後, 以下の通りにsourceを行う.

※ これを行わないと, buildを行った結果がrosシステムに反映されない.

```bash
source install/setup.bash
```

## 実行方法(2つのターミナルを使用)

1. ROS2(Linux PC)とUnity(DAIV)をLANケーブルで接続する

```bash
ros2 launch ros_tcp_endpoint endpoint.py
#注意：endpoint.pyはtab補完が効かない為手入力を行う
```
### Unity側での設定
- UnityEditorの上部ツールバーからRobotics->ROS Settingを開き, "ROS IP Address", "ROS Port"のところにROS側のIPアドレスおよびポート番号(default : 10000)を入力.
- "Protocol"が"ROS2"になっていることを確認.

<div align="center">
  <img width="600" alt="ROS-TCP-Connector" src="https://github.com/user-attachments/assets/5f94d2f4-8980-4eb5-bc0a-0676636a3a0c" />
  <br><br>
  <img width="600" alt="Connection_Statue" src="https://github.com/user-attachments/assets/e7a2ce7c-30a3-4f80-b7ed-09e40f96d42e" />
</div>


- 正常に接続されると, LinuxPCのターミナルにOKとlogが表示される.
  
`[default_server_endpoint-1] [INFO] [1782343553.791738689] [UnityEndpoint]: RegisterPublisher(clock, <class 'rosgraph_msgs.msg._clock.Clock'>) OK`


2. Linux PCとシミュレータPC間の通信ができたら, Linux PCで別のターミナルを立ち上げ以下を実行する.
```bash
ros2 launch task_manager hackathon_sample.launch.py
```

すると, zx200(ドラグショベル)が指定回数の掘削・積込動作を行い, ic120(クローラダンプ)が移動.
指定された放土位置に到着後, ベゼル(荷台)を上げ下げし放土を行う. 放土完了後, ic120は, はじめの位置に移動し, サンプルコードが終了する.

## ソフトウェアシステム

### hackathon_sample_manager_node
- 主な役割
システム全体の作業順序を管理する上位ノードである.
zx200とic120の作業完了通知を受信し, 現在の作業状態に応じて次の指令を送信する.

- 管理する作業
1. zx200の掘削開始
2. zx200の掘削完了待ち
3. ic120の排土場所への搬送
4. ic120の荷台上昇
5. ic120の荷台下降
6. ic120のホーム位置への帰還
7. 全作業完了

- 主なPublishトピック

|トピック|型|内容|
|:---|:---|:---|
|/start_dig|std_msgs/Bool|zx200の掘削開始|
|/ic120/start_transport|std_msgs/Bool|ic120の搬送開始|
|/ic120/tilt_up_cmd|std_msgs/Bool|ic120の荷台上昇|
|/ic120/tilt_down_cmd|std_msgs/Bool|ic120の荷台下降|
|/ic120/start_return|std_msgs/Bool|ic120の帰還開始|


- 主なSubscribeトピック

|トピック|型|内容|
|:---|:---|:---|
|/end_dig|std_msgs/Bool|zx200の掘削完了|
|/ic120/arrived_dump|std_msgs/Bool|ic120の排土場所到着|
|/ic120/dump_completed|std_msgs/Bool|排土動作完了|
|/ic120/tilt_down_completed|std_msgs/Bool|荷台下降完了|
|/ic120/arrived_home|std_msgs/Bool|ic120のホーム帰還完了|

- ノード間の責務分担

|機能|担当ノード|
|:---|:---|
|全作業の順序管理|hackathon_sample_manager_node|
|zx200の掘削動作|state_machine_node|
|ic120の往路・帰路走行|ic120_autonomy_node|
|ic120の荷台上昇・下降|dump_release_controller|
|ic120の経路生成・経路追従|Nav2|
|車両・荷台の物理動作|Unity|
|ROSとUnity間の通信|ROS-TCP Connector/Endpoint|

### 設計上の重要点
現在の構成では, task_managerは実際の関節や速度を直接制御しない.

task_manager

    ↓  作業開始指令

各機能ノード

    ↓  具体的な制御指令

Nav2 または Unity

という階層構造担っている.

特にic120では, 責務を次のように分離している.

ic120_autonomy_node ->  走行制御

dump_release_controller ->  荷台制御

task_manager-> 両者の作業順序を管理

この分離により, 走行制御と荷台制御の処理が競合することを防ぎ, 各機能を独立して試験できる構成となっている.

ros2 launch task_manager hackathon_launch.py実行時のノード/トピックパイプライン(rqt_graph)
<div align="center">
  <img width="600" alt="rosgraph" src="https://github.com/user-attachments/assets/2d5274f4-0f25-40da-a807-3020f55082b4" />
</div>

