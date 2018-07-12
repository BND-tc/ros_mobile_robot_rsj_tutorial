---
title: 点群処理とロボットナビゲーションの統合
date: 2018-07-13
---

- Table of contents
{:toc}

# 点群処理とロボットナビゲーションの統合

これまでにセミナーで取り扱った内容のおさらいに演習を用意していますので、取り組んでみてください。

# PointCloudのデータを使ったロボット制御
RViz 上で紫で表示されている、センサに最も近いクラスタに対してロボットが近づくように制御してください。ただし一定の距離まで近づいたら停止するようにしてください。

ヒント：ロボットローカル座標系における紫のクラスタの位置はすでに計算されています。ロボットがその方向を向くように角速度を算出します。直進速度は一定の値でも良いですし、角速度が大きくなる場合はそれに応じて小さくするなどすれば小回りが効く制御になります。

- [解答例](https://github.com/KMiyawaki/rsj_robot_answer/blob/answer_1/src/rsj_robot_test.cpp)

# URG-04LX-UG01(2DURG)
2DURG のデータを利用して、ロボットが人に追従するように制御してみましょう。基本的な方法は上記の課題と同じです。

- [解答例](https://github.com/KMiyawaki/rsj_robot_answer/blob/answer_2/src/rsj_robot_test.cpp)

# ROS navigation メタパッケージの利用
RViz 上で紫で表示されている、センサに最も近いクラスタに対してロボットが近づくように制御してください。ただし、ROS navigation メタパッケージを利用します。これにより、経路上の障害物を回避しながら近づくようにします。

ヒント：
- `/move_base_simple/goal`トピックに`geometry_msgs::PoseStamped`型のメッセージをパブリッシュし、目標地点の位置姿勢を与えることで、ナビゲーションの指示を与えられます。
- `geometry_msgs::PoseStamped`型のメッセージの作成方法の例は次のとおりです。

```c++
geometry_msgs::PoseStamped goal;
goal.header.frame_id = "base_link"; // ロボットローカル座標におけるゴール位置を指定したい場合のコード。

goal.pose.position.x = 目標の x 座標; 
goal.pose.position.y = 目標の y 座標;

// 目標地点に到達した際にロボットを向かせたい方向を指定する。
goal.pose.orientation = tf::createQuaternionMsgFromYaw(ロボットから見た目標の方向をラジアンで指定); 
// 良くわからなければゼロ度でも構わない。
```

- `geometry_msgs::PoseStamped`のパブリッシュは目標発見時に1回行えば十分です。したがって、目標探索中か追跡中かといった状態を管理するのが簡単です。

- [解答例](https://github.com/KMiyawaki/rsj_robot_answer/blob/answer_3/src/rsj_robot_test.cpp)
