---
title: ROSの基本操作
date: 2018-04-10
---

- Table of contents
{:toc}

基本的なROS上で動くプログラムの書き方とビルド方法を学習します。

## 基本的な用語

パッケージ
: ノードや設定ファイル、コンパイル方法などをまとめたもの

ノード
: ROSの枠組みを使用する、実行ファイル

メッセージ
: ノード間でやりとりするデータ

トピック
: ノード間でメッセージをやりとりする際に、メッセージを置く場所

ノード、メッセージ、トピックの関係は以下の図のようにに表せます。

![ROS topics](images/ros-topic.png)

基本的には、ソフトウェアとしてのROSは、ノード間のデータのやりとりをサポートするための枠組みです。加えて、使い回しがきく汎用的なノードを、世界中のROS利用者で共有するコミュニティも、大きな意味でのROSの一部となっています。

## ソースコードを置く場所

ROSでは、プログラムをビルドする際に、catkin というシステムを使用しています。また、catkin は、 cmake というシステムを使っており、ROS用のプログラムのパッケージ毎に、cmakeの設定ファイルを作成することで、ビルドに必要な設定を行います。

以下の手順で本作業用の新しいワークスペースを作ります。

```shell
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/src
$ catkin_init_workspace
Creating symlink "/home/[ユーザディレクトリ]/catkin/src/CMakeLists.txt"
    pointing to "/opt/ros/kinetic/share/catkin/cmake/toplevel.cmake"
$ ls
CMakeLists.txt
$ cd ..
$ ls
src
$
```

`catkin_ws`ディレクトリ内にある、`build`、`devel`は、catkinシステムがプログラムをビルドする際に使用するものなので、ユーザが触る必要はありません。`catkin_ws/src`ディレクトリは、ROSパッケージのソースコードを置く場所で、中にある`CMakeLists.txt` は、ワークスペース全体をビルドするためのルールが書かれているファイルです。

このディレクトリに、ypspur-coordinatorをROSに接続するためのパッケージypspur_rosをダウンロードします。

```shell
$ cd ~/catkin_ws/src
$ git clone https://github.com/openspur/ypspur_ros.git
$ ls
CMakeLists.txt  ypspur_ros
$
```

gitは、ソースコードなどの変更履歴を記録して管理する、分散型バージョン管理システムと呼ばれるものです。今回のセミナーでは詳細は触れませんが、研究開発を行う上では非常に有用なシステムですので、利用をお勧めします。公式の解説書、[Pro Git](https://git-scm.com/book/ja/v2")などを参考にして下さい。

GitHubは、ソースコードなどのリポジトリーサービスです。オープンソースソフトウェアの開発、共同作業及び配布を行うためによく利用されて、ROSではソースコードの保存と配布する場所としてもっとも人気なサービスです。バイナリーパッケージとして配布されているROSパッケージ以外の利用をする場合、GitHubを利用します。URLが分かれば上の手順だけで簡単にROSのパッケージを自分のワークスペースにインポートし利用することができます。

では、次にパッケージのディレクトリ構成を確認します。ダウンロードしているパッケージがバージョンアップされている場合などには、下記の実行例とファイル名が異なったり、ファイルが追加・削除されているが場合があります。

```shell
$ cd ypspur_ros/
$ ls
CMakeLists.txt  msg  package.xml  src
$ ls msg/
ControlMode.msg  DigitalOutput.msg  JointPositionControl.msg
$ ls src/
getID.sh  joint_tf_publisher.cpp  ypspur_ros.cpp
$
```

`CMakeLists.txt`と`package.xml`には、使っているライブラリの一覧や、生成する実行ファイルとC++のソースコードの対応など、このパッケージをビルドするために必要な情報が書かれています。`msg`ディレクトリには、このパッケージ独自のデータ形式の定義が、`src`ディレクトリには、このパッケージに含まれるプログラム(ノード)のソースコードが含まれています。

次に`catkin_make`コマンドで、ダウンとロードした`rsj_tutorial_2017_ros_intro`パッケージを含む、ワークスペース全体をビルドします。`catkin_make`は、ワークスペースの最上位ディレクトリ(`~/catkin_ws/`)で行います。

```shell
$ cd ~/catkin_ws/
$ catkin_make
・・・実行結果・・・
```

## ROSノードの理解とビルド・実行

端末を開き、ひな形をダウンロードします。

```shell
$ cd ~/catkin_ws/src/
$ git clone https://github.com/at-wat/rsj_robot_test.git
```

ソースファイルの編集にはお好みのテキストエディターが利用可能です。本セミナーの説明ではメジャーなテキストエディタである`emacs` を用います。Linuxがはじめての方には`gedit`もおすすめです。

お好みのテキストエディターで`~/catkin_ws/src/rsj_robot_test.cpp`を開きます。

![greeter.cpp](images/greeter_cpp_in_editor.png)

### 基本的なコードを読み解く

このコードが実行されたときの流れを確認しましょう。

まず、先頭部分では、必要なヘッダファイルをインクルードしています。

```c++
#include <ros/ros.h>
```

続いて、rsj_robot_test_nodeクラスを定義しています。ROSプログラミングの際には、基本的にノードの持つ機能をクラスとして定義し、これを呼び出す形式を取ることが標準的です。(クラスを使用せずに書く事も可能ですが、気をつけなければならない点が多くなるため、本セミナーではクラスでの書き方のみを解説します。)

```c++
class rsj_robot_test_node
{
 (略)
public:
 (略)
	void mainloop()
	{
		ROS_INFO("Hello ROS World!");

		ros::Rate rate(10.0);
		while(ros::ok())
		{
			ros::spinOnce();
			// ここに速度指令の出力コード
			rate.sleep();
		}
	}
};
```

rsj_robot_test_nodeクラスのメンバ関数であるmainloop関数の中では、ROSで情報を画面などに出力する際に用いる、ROS_INFO関数を呼び出して、"Hello ROS World!"と表示しています。ほかにも、ROS_DEBUG、ROS_WARN、ROS_ERROR、ROS_FATAL関数が用意されています。

ros::Rate rate(10.0)で、周期実行のためのクラスを初期化しています。初期化時の引数で実行周波数(この例では10Hz)を指定します。

while(ros::ok())で、メインの無限ループを回します。ros::ok()をwhileの条件にすることで、ノードの終了指示が与えられたとき(Ctrl+Cが押された場合も含む)には、ループを抜けて終了処理などが行えるようになっています。

ループ中では、まず、ros::spinOnce()を呼び出して、ROSのメッセージを受け取るといった処理を行います。spinOnceは、その時点で届いているメッセージの受け取り処理を済ませた後、すぐに処理を返します。rate.sleep()は、先ほど初期化した実行周波数を維持するようにsleepします。

なお、ここでは、クラスを定義しただけなので、中身が呼び出されることはありません。後ほど実体化されたときに、初めて中身が実行されます。

続いて、C++のmain関数が定義されています。ノードの実行時には、ここから処理がスタートします。

```c++
int main(int argc, char **argv) {
	ros::init(argc, argv, "rsj_robot_test_node");
	
	rsj_robot_test_node robot_test;
	
	robot_test.mainloop();
}
```
はじめに、ros::init関数を呼び出して、ROSノードの初期化を行います。1、2番目の引数には、main関数の引数をそのまま渡し、3番目の引数には、このノードの名前(この例では"rsj_robot_test_node")を与えます。

次に、rsj_robot_test_nodeクラスの実体を作成します。ここでは、robot_testと名前をつけています。

最後に、実体化したrobot_testのメンバ関数、mainloopを呼び出します。mainloop関数の中は無限ループになっているため、終了するまでの間、ros::spinOnce()、rate.sleep()が呼び出され続けます。

つまり、rsj_robot_testは特に仕事をせず、"Hello ROS World!"と画面に表示します。

### ビルド＆実行

ROS上でこのパッケージをビルドするためには、`catkin_make`コマンドを用います。

```shell
$ cd ~/catkin_ws/
$ catkin_make
```

端末で実行してみましょう。

ROSシステムの実行の際、ROSを通してノード同士がデータをやりとりするために用いる、「roscore」を起動しておく必要があります。2つ目の端末を開き、それぞれで以下を実行して下さい。

1つ目の端末：

```shell
$ roscore
```

ROSでワークスペースを利用するとき、端末でそのワークスペースをアクティベートすることが必要です。このためにワークスペースの最上のディレクトリで`source devel/setup.bash`を実行します。このコマンドはワークスペースの情報を利用中の端末に読み込みます。しかし、 _仮のことだけ_ ので必ず新しい端末でワークスペースを利用し始めると _必ず_{: style="color: red"} まずは`source devel/setup.bash`を実行しなければなりません。一つの端末で一回だけ実行すれば十分です。その端末を閉じるまでに有効です。

2つ目の端末で下記を実行します。

```shell
$ rosrun rsj_robot_test rsj_robot_test_node
[ INFO] [1466002781.136800000]: Hello ROS World!
```

「Hello ROS World!」と表示されれば成功です。以上の手順で、ROSパッケージに含まれるノードのソースコードを編集し、ビルドして、実行できるようになりました。

両方の端末で __Ctrl+c__{: style="border: 1px solid black" } でノードと`roscore`を終了します。

### ロボットに速度指令を与える

まず、ロボットに速度指令(目標並進速度・角速度)を与えるコードを追加します。ひな形には既に、速度指令値が入ったメッセージを出力するための初期化コードが含まれていますので、この部分の意味を確認します。

```c++
rsj_robot_test_node():
{
	ros::NodeHandle nh("~");
	pub_twist = nh.advertise<geometry_msgs::Twist>(
			"/ypspur_ros/cmd_vel", 5);
	sub_odom = nh.subscribe("/ypspur_ros/odom", 5,
			&rsj_robot_test_node::cb_odom, this);
}
```

ソースコード中の、`rsj_robot_test_node`クラスの、`rsj_robot_test_node`関数は、クラスのコンストラクタと呼ばれるもので、クラスが初期化されるときに自動的に呼び出されます。
この中で、

```c++
nh.advertise<geometry_msgs::Twist>("/ypspur_ros/cmd_vel", 5);
```

の部分で、このノードが、<u>これからメッセージを出力する</u>ことを宣言しています。`advertise`関数に与えている引数は以下のような意味を持ちます。

`"/ypspur_ros/cmd_vel"`
: 出力するメッセージを置く場所(トピックと呼ぶ)を指定

`5`
: メッセージのバッファリング量を指定 (大きくすると、処理が一時的に重くなったときなどに受け取り側の読み飛ばしを減らせる)

`advertise` 関数についている、 `<geometry_msgs::Twist>` の部分は、メッセージの型を指定しています。これは、幾何的・運動学的な値を扱うメッセージを定義している `geometry_msgs` パッケージの、並進・回転速度を表す `Twist` 型です。(この指定方法は、C++のテンプレートという機能を利用していますが、ここでは、「 `advertise` のときはメッセージの型指定を `< >` の中に書く、とだけ覚えておけば問題ありません。)

以下のコードを、 `mainloop` 関数の中(「ここに速度指令の出力コード」の部分)に入れることで、速度指令のメッセージを出力( `publish` )します。

```c++
geometry_msgs::Twist cmd_vel;
cmd_vel.linear.x = 0.05;
cmd_vel.angular.z = 0.0;
pub_twist.publish(cmd_vel);
```

### ビルド＆実行
```shell
$ cd ~/catkin_ws/
$ catkin_make
```
この際、ビルドエラーが出ていないか、良く確認して下さい。エラーが出ている場合は、ソースコードの該当箇所を確認・修正して下さい。

実行の際、まず `roscore` と、 `ypspur_ros` を起動します。 `ypspur_ros` の中では、ロボットの動作テストの際に使用した、 `ypspur-coordinator` が動いています。なお、 `roscore` は、前のものを実行し続けている場合は、そのままで使用できます。コマンド入力の際は、タブ補完を活用しましょう。

```shell
$ roscore
```

2番目の端末を開いて、下記を実行します。

```shell
$ rosrun ypspur_ros ypspur_ros _param_file:=/home/ubuntu/params/rsj-seminar20??.param <u>該当するものに置き換えること</u> _port:=/dev/serial/by-id/usb-T-frog_project_T-frog_Driver-if00
```

ゆっくりとホイールが回れば、正しく動作しています。Ctrl+Cで終了します。

### 少課題

速度、角速度を変更して動作を確認してみましょう。

## ロボットの状態を表示する
### ロボットの状態を表示するコードを追加

まず、ロボットの動作したときの移動量やオドメトリ座標を取得、表示するコードを追加します。ひな形には既に、移動量や座標が入ったメッセージを受け取るコードが含まれていますので、この部分の意味を確認します。

```c++
rsj_robot_test_node():
{
	ros::NodeHandle nh("~");
	pub_twist = nh.advertise<geometry_msgs::Twist>(
			"/ypspur_ros/cmd_vel", 5);
	sub_odom = nh.subscribe("/ypspur_ros/odom", 5,
			&rsj_robot_test_node::cb_odom, this);
}
```

この中で

```c++
nh.subscribe("/ypspur_ros/odom", 5, &rsj_robot_test_node::cb_odom, this);
```

の部分で、このノードが、これからメッセージを受け取ることを宣言しています。subscribe関数に与えている引数は以下のような意味を持ちます。

`"/ypspur_ros/odom"`
: 受け取るメッセージが置かれている場所(トピック)を指定

`5`
: メッセージのバッファリング量を指定 (大きくすると、処理が一時的に重くなったときなどに受け取り側の読み飛ばしを減らせる)

`&rsj_robot_test_node::cb_odom`
: メッセージを受け取ったときに呼び出す関数を指定 (rsj_robot_test_nodeクラスの中にある、cb_odom関数)

`this`
: メッセージを受け取ったときに呼び出す関数がクラスの中にある場合にクラスの実体を指定 (とりあえず、おまじないと思って構いません。)

これにより、rsj_robot_test_nodeノードは、/odomトピックからメッセージをうけとると、cb_odom関数が呼び出されるようになります。続いてcb_odom関数の中身を確認しましょう。

```c++
void cb_odom(const nav_msgs::Odometry::ConstPtr &msg)
{
}
```

const nav_msgs::Odometry::ConstPtr は、const型(内容を書き換えられない)、nav_msgsパッケージに含まれる、Odometry型のメッセージの、const型ポインタを表しています。&msgの&は、参照型(内容を書き換えられるように変数を渡すことができる)という意味ですが、(const型なので)ここでは特に気にする必要はありません。

cb_odom関数に、以下のコードを追加してみましょう。これにより、受け取ったメッセージの中から、ロボットの並進速度を取り出して表示できます。

```c++
ROS_INFO("vel %f", msg->twist.twist.linear.x);
```

ここで、msg->twist.twist.linear.x の意味を確認します。nav_msgs::Odometryメッセージには、下記のように入れ子状にメッセージが入っています。

```shell
std_msgs/Header header
string child_frame_id
geometry_msgs/PoseWithCovariance pose
geometry_msgs/TwistWithCovariance twist
```

全て展開すると、以下の構成になります。

```shell
std_msgs/Header header
uint32 seq
time stamp
string frame_id
string child_frame_id
geometry_msgs/PoseWithCovariance pose
geometry_msgs/Pose pose
geometry_msgs/Point position
float64 x
float64 y
float64 z
geometry_msgs/Quaternion orientation
float64 x
float64 y
float64 z
float64 w
float64[36] covariance
geometry_msgs/TwistWithCovariance twist
geometry_msgs/Twist twist
geometry_msgs/Vector3 linear
float64 x ロボット並進速度
float64 y
float64 z
geometry_msgs/Vector3 angular
float64 x
float64 y
float64 z ロボット角速度
float64[36] covariance
```

読みたいデータである、ロボット並進速度を取り出すためには、これを順にたどっていけば良く、msg->twist.twist.linear.x、となります。msgはクラスへのポインタなので「->」を用い、以降はクラスのメンバ変数へのアクセスなので「.」を用いてアクセスしています。

### ビルド＆実行
```shell
$ cd ~/catkin_ws/
$ catkin_make
```

この際、ビルドエラーが出ていないか、良く確認して下さい。エラーが出ている場合は、ソースコードの該当箇所を確認・修正して下さい。

まず、先ほどと同様、roscoreと、ypspur_rosを起動します。(以降、この手順の記載は省略します。)

```shell
$ roscore
$ rosrun ypspur_ros ypspur_ros _param_file:=/home/ubuntu/params/rsj-seminar20??.param <u>該当するものに置き換えること</u> _port:=/dev/serial/by-id/usb-T-frog_project_T-frog_Driver-if00
```

続いて、rsj_robot_test_nodeノードを実行します。

```shell
$ rosrun rsj_robot_test rsj_robot_test_node
Hello ROS World!
vel: 0.0500
vel: 0.0500
vel: 0.0500
vel: 0.0500
```

ロボットのホイールが回転し、先ほどの小課題で設定した走行指令の値と近い値が表示されれば、正しく動作しています。

### 小課題
同様に、ロボットの角速度を表示してみましょう。

## シーケンス制御
時間で動作を変える
メインループを以下のように変更してみましょう。

```c++
void mainloop()
{
	ROS_INFO("Hello ROS World!");

	ros::Rate rate(10.0);
	ros::Time start = ros::Time::now();
	while(ros::ok())
	{
		ros::spinOnce();
		ros::Time now = ros::Time::now();

		geometry_msgs::Twist cmd_vel;
		if(now - start > ros::Duration(3.0))
		{
			cmd_vel.linear.x = 0.05;
			cmd_vel.angular.z = 0.0;
		}
		pub_twist.publish(cmd_vel);

		rate.sleep();
	}
}
```

これは、メインループ開始時刻から、3.0秒後に、並進速度0.05m/sの指令を与えるコードです。ros::Time型(時刻を表す)同士の減算結果は、ros::Duration型(時間を表す)になり、比較演算子で比較できます。したがって、now - start > ros::Duration(3.0)の部分は、開始から3秒後に、trueになります。

先ほどと同様にビルドし、ypspur_rosとrsj_robot_test_nodeを起動して動作を確認します。

センシング結果で動作を変える
cb_odomで取得したオドメトリのデータを保存しておくように、以下のように変更してみましょう。(赤線部分を追加)

```c++
void cb_odom(const nav_msgs::Odometry::ConstPtr &msg)
{
	ROS_INFO("vel %f", 
		msg->twist.twist.linear.x);
	odom = *msg;
}
```

また、class rsj_robot_test_nodeの先頭に下記の変数定義を追加します。

```c++
class rsj_robot_test_node
{
private:
	nav_msgs::Odometry odom;
```

また、odomの中で方位を表す、クオータニオンをコンストラクタ(rsj_robot_test_node()関数)の最後で初期化しておきます。

```c++
rsj_robot_test_node():
{
	(略)
	odom.pose.pose.orientation.w = 1.0;
}
```

メインループを以下のように変更してみましょう。

```c++
void mainloop()
{
	ROS_INFO("Hello ROS World!");

	ros::Rate rate(10.0);
	while(ros::ok())
	{
		ros::spinOnce();

		geometry_msgs::Twist cmd_vel;
		if(tf::getYaw(odom.pose.pose.orientation) > 1.57)
		{
			cmd_vel.linear.x = 0.0;
			cmd_vel.angular.z = 0.0;
		}
		else
		{
			cmd_vel.linear.x = 0.0;
			cmd_vel.angular.z = 0.1;
		}
		pub_twist.publish(cmd_vel);

		rate.sleep();
	}
}
```

これは、オドメトリのYaw角度(旋回角度)が1.57ラジアン(90度)を超えるまで、正方向に旋回する動作を表しています。

先ほどと同様にビルドし、ypspur_rosとrsj_robot_test_nodeを起動して動作を確認します。

### 小課題

1m前方に走行し、その後で帰ってくるコードを作成してみましょう。(1m前方に走行し180度旋回して1m前方に走行するか、もしくは、1m前方に走行し1m後方に走行すればよい。)

余裕があれば、四角形を描いて走行するコードを作成してみましょう。