---
title: 画像処理とOpenCVの利用
date: 2017-06-16
---

本セクションでは、前セクションで取得した画像を処理する方法について説明します。特にOpenCVを用いて処理する方法について説明します。

本セクションでは、画像に基づきブロックを見つけ、その位置を出力する一連の処理について説明します。

# OpenCV

OpenCV（Open Source Computer Vision Library）は無料の画像処理ライブラリーです。Linuzの他、WiondowsやMacOSでも利用することができ、現在、多くの画像処理研究で利用されてます。例えば、OpenCVを利用することで、従来手法との精度比較を簡単に行うことができます。

ROSでOpenCVを利用するときの注意点としては、バージョン管理があります。ROSがリリースされたときの最新バージョンのOpenCVを使用することになります。ROSのバージョンとOpenCVのバージョンの対応表をまとめておきます。本セミナーはROS16.04を使用しているため、OpenCV3を利用することになります。

|ROSのバージョン|OpenCVのバージョン|
|17.04 (Lunar Loggerhead)|3.2.0|
|16.04 (Kinetic Kame)|3.1.0|
|15.04 (Jade Turtle)|2.4.11|
|14.04 (Indigo Igloo)|2.4.8|

# OpenCVの設定

1. まず、OpenCVをインストールします。

   ```shell
   sudo apt-get install ros-kinetic-vision-opencv 
   sudo apt-get install python-opencv
   sudo apt-get install libopencv-dev
   ```

1. 次に、OpenCVと正しくmakeできるようにCMakeLists.txtを修正します。

   > find_package(OpenCV REQUIRED)
   
   > include_directories(/usr/local/include ${catkin_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} )
   
   > target_link_libraries(dfollow ${catkin_LIBRARIES} ${OpenCV_LIBRARIES} )

1. また、package.xmlも修正します。OpenCV3を使用しますが、互換性を保つためにopencv2と指定します。

   > `<build_depend>opencv2</build_depend>`

   > `<run_depend>opencv2</run_depend>`

1. 本セミナーではパッケージ『cv_bridge』を利用します。このパッケージはROSの画像データ（Image Message）とOpenCVの画像データ（IplImage）を相互に変換することができます。つまり、IplImageへ変換し、処理を施し、Image Messageへ戻すという一連の処理を記述することができます。（※IplはIntel Image Processing Libraryの略で、バージョン1で使用されている型になります。そのため、本セミナーではIplImageをバージョン2以降で使用されている型『Mat』へ変換し、画像処理を行います。）

   ```shell
   sudo apt-get install ros-kinetic-cv-camera
   ```

# 画像処理パッケージの設定

1. セミナー用画像処理のROSパッケージをダウンロードします。ディレクトリ『rsj_2017_block_finder』が存在することを確認します。

   ```shell
   $ cd ~/block_finder_ws/src
   $ git clone git@github.com:Suzuki1984/rsj_2017_block_finder.git
   $ ls
   CMakeLists.txt  rsj_2017_block_finder  usb_cam
   ```

1. コンパイルします。エラーが出ず、[100%]となることを確認します。

   ```shell
   $ cd ~/block_finder_ws/
   $ catkin_make 
   ```

1. ワークスペース内のROSパッケージが利用できるようにワークスペースをソースします。

   ```shell
   $ source devel/setup.bash
   ```

これでセミナー用画像処理のパッケージ「rsj_2017_block_finder」が利用可能になりました。


# カメラのキャリブレーション

カメラの内部パラメーターを設定します。

1. チェッカーボードの頂点の数と大きさを確認します。このとき、四角形の数を数えないように注意してください。また、長さの単位がメートルであることにも注意してください。

1. ROSパッケージ『camera_calibration』をインストールします。

   ```shell
   $ sudo apt-get install ros-kinetic-camera-calibration
   ```

1. ROSパッケージ『camera_calibration』を実行します。

   ```shell
   # １つ目のターミナル
   $ roslaunch rsj_2017_block_finder usb_cam_calib.launch
   # ２つ目のターミナル
   $ rosrun camera_calibration cameracalibrator.py --size 8x6 --square 0.0285 image:=/camera/image_raw camera:=/camera
   ```

1. カメラの位置や姿勢を動かします。（あるいはチェッカーボードを上下左右や遠近に移動したり傾けたりします。）
1. X, Y, Sizeが全て右端まで伸びたらCALIBRATEボタンを押します。
1. チェッカーボードの直線が画面上でも直線になっていることを確認します。
1. COMMITボタンを押します。


# 基礎編

1. まず、セミナー用画像処理パッケージの内容を確認します。

   ```shell
   $ cd rsj_2017_block_finder
   $ ls
   CMakeLists.txt  config  launch  package.xml  readme.md  src
   ```

1. ディレクトリ『launch』の中にはblock_finder.launchとblock_finder_w_stp.launchがあります。ここでは後者のファイルを確認します。後者のlaunchファイルでは4つのノードを起動します。配信（publish）と購読（listen）の関係性を以下に示します。

   > <?xml version="1.0"?>
   > <launch>
   >   <node pkg="usb_cam" type="usb_cam_node" name="camera" output="screen">
   >     <param name="camera_name" value="elecom_ucam"/>
   >     <param name="camera_frame_id" value="camera_link"/>
   >     <param name="video_device" value="/dev/video0"/>
   >     <param name="image_width" value="640"/>
   >     <param name="image_height" value="480"/>
   >     <param name="pixel_format" value="yuyv"/>
   >     <param name="io_method" value="mmap"/>
   >   </node>
   >   <node pkg="tf" type="static_transform_publisher" name="camera_transform_publisher" args="0 0 0 0 0 0 1 /world /camera_link 1"/>
   >   <node pkg="rsj_2017_block_finder" type="block_finder" name="block_finder" args="$(arg method)" output="screen"/>
   >   <node pkg="rviz" type="rviz" name="rviz" args="-d $(find rsj_2017_block_finder)/config/rsj_2017_block_finder.rviz"/>
   > </launch>

   `camera`
   : 画像メッセージを配信します。

   `camera_transform_publisher`
   : World座標系の原点から見たCamera座標系の原点の位置を配信します。

   `block_finder`
   : 画像メッセージを購読し、処理し、World座標系におけるブロックの位置を配信します。

   `rviz`
   : World座標系、Camera座標系、ブロックの位置を表示します。

## 画像処理プログラムの概要

カメラを接続し、カメラのデバイス番号を確認します。デバイス番号が0以外の場合はlaunchファイルを修正してください。

チェッカーボードを机の上に置いたあと、下記のコマンドで実行します。カラー画像、グレー画像、RVizの３つの画面が開きます。チェッカーボード上に黄色の四角形が表示されていれば正常に起動しています。

```shell
$ roslaunch rsj_2017_block_finder block_finder_w_stp.launch method:=1
```

![Block Finder GUI](images/block_finder_area.png)

チェッカーボードが見つかると、ターミナルに「Camera座標系からBoard座標系までの変換ベクトル」が表示されます。３番目の値（Ｚ軸の値）の正負を置き換えた値をstatic_transform_publisherの３番目の値にすると結果が確認しやすくなります。例えば、次のとおりに設定します。

> args="0 0 -0.478 0 0 0 1 /world /camera_link 1"

次に、チェッカーボードの上に厚紙を置いたりチェッカーボードを退かしたりします。

そして、黄色の四角形の中に収まるようにブロックを置きます。

ここでRVizを確認しましょう。TFは座標系を表示し、R色がX軸、G色がY軸、B色がZ軸を表します。「Global　Options」→「Fixed Frame」で基準とするFrameを切り替えることができます。 なお、PointStampedはHeaderとPointが組み合わさったメッセージで、Headerで位置データを取得した時刻、Pointで位置データを表現することができます。

ここでは、上述のとおり、World座標系からCamera座標系までの変換ベクトルを適当に与えています。最後のセクションではROSパッケージ『crane_plus_camera_calibration』を利用して同ベクトルを求め、マニピュレーターがブロックを正しく把持できるようにします。

## 画像処理法

ブロックを検出するための画像処理について見ていきます。

まず、関数『GaussianBlur』で平滑化を行います。平滑化を行うことで、後述の２値化処理が安定します。第３引数ではフィルタのサイズを指定することができ、cv::Size(5, 5)やcv::Size(13, 13)などと、正の奇数で指定します。
   
それでは、実際に値を変更してみましょう。値を変更し、ファイルを上書きしたら、下記のとおりコンパイルし、実行します。
   
   ```shell
   $ cd ~/block_finder_ws/
   $ catkin_make
   $ roslaunch rsj_2017_block_finder block_finder.launch method:=1
   ```
   
次に、関数『threshold』で２値化します。第３引数が閾値となり、この閾値を境に各ピクセルに０と１の値を与えていきます。本セミナーではトラックバーを使用して動的に変更できるようにしてあります。トラックバーを直接ドラッグするほか、トラックバーの左右をクリックすることで５刻みで値を増減することもできます。

そして、関数『findContours』を使用してブロックを認識します。第３引数では近似手法を指定することができ、現在はCV_CHAIN_APPROX_NONEとなっています。CV_CHAIN_APPROX_SIMPLEやCV_CHAIN_APPROX_TC89_L1に変更し、結果の違いを確認してみてください。
   
- CV_CHAIN_APPROX_NONE
  : 全ての点を保存します。
- CV_CHAIN_APPROX_SIMPLE
  : 端点のみを保存します。つまり、輪郭を表現する点群を圧縮します。
- CV_CHAIN_APPROX_TC89_L1
  : Teh-Chinアルゴリズムで、NONEとSIMPLEの中間に当たる。

## 画像の表示

OpenCVでは関数『inshow』を使用して画像を表示します。関数『imshow』のあとに関数『waitKey』を呼び出することで、画像が表示されます。関数『waitKey』は一定時間キー入力を待つ関数ですが、ここではスリープ関数のような意味を持ちます。なお、繰り返し処理でない画像処理の場合、０として指定することで、キー入力が行われるまで画像を表示しておくことができます。
   
なお、ウィンドウの名前は関数『namedWindow』で、ウィンドウの位置は関数『moveWindow』で指定することができます。
   
デストラクタ『~BlockFinder』の中に関数『destroyWindow』を記述しておくことで、メモリの開放忘れを予防することができます。関数『destroyAllWindows』もあります。

## ブロック位置の出力

２次元画像上でブロックの位置を認識したあとは、下図のとおりWorld座標系での位置へ変換し、Publishする。

![Block Finder GUI](images/block_finder_transform.png)

OpenCVの関数『projectPoints』を利用することで、２次元画像上の位置とボードの左上を原点とした３次元空間（target_frame）上の位置の対応関係を得ることができる。

次に、tfの関数『transformPoint』を利用することで、ボード座標系（target_frame）の位置を、Camera座標系（camera_frame）を経由して、World座標系（fixed_frame）の位置へ変換する。

最終的にPublishされている３次元座標値をコマンド『rostopic echo』を使用して確認します。別のターミナルを開いて、下記のとおり実行します。


```shell
$ rostopic echo /block_finder/pose
```

なお、２次元画像上の位置は下記のとおり実行することで確認できます。


```shell
$ rostopic echo /block_finder/pose_image
```

また、ブロック領域の面積を下記のとおり実行することで確認できます。このプログラムでは、大きすぎる場合や小さすぎる場合はPublishしないようにしています。

```shell
$ rostopic echo /block_finder/block_size_max
```

『Ctrl』キー＋『c』キーで終了します。

# 発展編

基本編で使用した輪郭検出処理では、チェッカーボードの四角形をスポンジと誤認識してしまいます。そのため、チェッカーボードの上でもスポンジを検出できるように改良します。

## 背景差分法

   本セミナーでは背景差分（特に動的背景差分）を利用します。OpenCVでは下記の手法が実装されています。

   - 混合正規分布法（MoG：Mixture of Gaussian Distribution）
       - createBackgroundSubtractorMOG2
   - k近傍法（kNN：k-nearest neighbor）
       - createBackgroundSubtractorKNN

   また、OpenCVにはopencv_contribという追加モジュール群が存在します。このモジュールにもインストールすることで下記の手法も使用することができます。

   - ベイズ推定法（GMG: Godbehere、Matsukawa、Goldberg）
       - createBackgroundSubtractorGMG

   OpenCVはバージョンが変わると、記述方法や機能が大幅に変更されます。例えば、2から3へバージョンが変わったときは、KNNなどが追加されましたが、GMGなどはcontribへ移動されました。注意してください。

## 混合正規分布法

   createBackgroundSubtractorMOG2は下記のとおり３つの引数を指定することができます。

   > Ptr<BackgroundSubtractorMOG2> cv::createBackgroundSubtractorMOG2	(int history = 500, double varThreshold = 16, bool detectShadows = true)

   第１引数では、過去何フレームまでを分布推定（モデル推定）に利用するかを指定することができる。

   第２引数では、各ピクセルが背景モデルに含まれるかどうかを判断するための閾値を指定することができる。

   第３引数では、影の影響を考慮するかどうかを指定することができる。trueにすると計算速度が若干低下するが、精度を向上することができる。

   それでは、下記の部分を修正し、結果の違いを確認してみましょう。
   
   > pMOG2 = cv::createBackgroundSubtractorMOG2();

   > pMOG2 = cv::createBackgroundSubtractorMOG2(1000);

   > pMOG2 = cv::createBackgroundSubtractorMOG2(1000, 8);
   
   roslaunchを実行するときにmethod:=2と変更し、実行してください。これにより、ROSパラメータ『method』が2に上書きされた状態でプログラムを実行することができます。

   ```shell
   $ cd ~/block_finder_ws/
   $ catkin_make
   $ roslaunch rsj_2017_block_finder block_finder.launch method:=2
   ```

# 参考情報

OpenCVには多くのサンプルプログラムが用意されており、研究初期の検討段階において、様々な手法を試すことができます。そして、同サンプルプログラムをROSノード化したROSパッケージ『opencv_apps』があります。

1. インストールは下記のとおり行います。

   ```shell
   $ sudo apt-get install ros-kinetic-opencv-apps
   ```

1. 画像内から円形を抽出するサンプルプログラムは下記のとおり実行します。

   ```shell
   $ roslaunch opencv_apps hough_circles.launch image:=/usb_cam_node/image_raw
   ```

1. 画像内から人間の顔を抽出するサンプルプログラムは下記のとおり実行します。

   ```shell
   $ roslaunch opencv_apps face_detection.launch image:=/usb_cam_node/image_raw
   ```

以上
