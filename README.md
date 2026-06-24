# Opera-Sim ハッカソンサンプルコード実行手順

## 使用パッケージ
- ros_tcp_endpoint
- ic120_ros2
- task_manager
- zx200_autonomy
- ic120_autonomy
各Githubのリンクを貼り付ける

## build手順
bash```
$ cd ~/ros2_ws/src
```

bash```
$ colcon build --packages-select ic120_navigation task_manager zx200_autonomy ic120_autonomy ros_tcp_endpoint
```

bash```
$ source install/setup.bash
```

## 実行方法(2つのターミナルを使用)
bash```
$ ros2 launch ros_tcp_endpoint endpoint.py
#注意：endpoint.pyはtab補完が効かない為手入力を行う
```

bash```
ros2 launch task_manager hackathon_launch.py
```

