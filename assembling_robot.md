---
title: ロボットの組立てと動作テスト
date: 2018-06-16
---

- Table of contents
{:toc}

# ロボットの組立てと動作テスト

教材のロボットハードウェアを組み立て、簡単な動作テストを行います。

# モータドライバのバスパワー化

本セミナーの教材として使用するモータドライバは、初期状態ではセルフパワー(USBケーブルの他に、別途電源を供給する必要のある)USBデバイスとして動作します。本セミナーでは、簡易的な使用のため、モータドライバに配線を追加してバスパワーUSBデバイスとして使用します。

1. 被覆電線を約80mmの長さで切断

1. ワイヤストリッパを用い、電線の両端5mmの被覆を剥く

1. モータドライバ上の「+5Vin」と「USB5V」の穴に電線を差し込み、半田付け  
  ![](images/bus-power.jpg)

1. 半田付けがうまく行えているか確認 (オーガナイザーまたはTAに声をかけてください)  
  _注意：「CON1」から5Vを供給して利用する場合には、ここで半田付けした電線を切除してください。そのままで「CON1」から5Vを供給すると、モータドライバおよびUSBで接続しているPCが破損する可能性があります。_{:style="color: red"}

# ロボットの組立て

1.  天板にモータマウントをネジ止め  
  表からM5のネジを軽く留める  
  ![](images/assembly_mount1.jpg)  
  同様にもう一本もネジ留め  
  ![](images/assembly_mount2.jpg)  
  反対側のモータマウントも同様に取り付け  
  ![](images/assembly_mount3.jpg)  
  モータマウント裏側の溝に、M5のナットをはめ込み  
  ![](images/assembly_mount4.jpg)

1.  モータマウントにモータをネジ止め  
  モータをモータマウントにはめ込み  
  ![](images/assembly_motor1.jpg)  
  モータマウント側面の穴から、M3のネジ _（ネジ頭が小さく、ワッシャーが入っているもの）_{: style="color: red" } 4本でモータを固定  
  ![](images/assembly_motor2.jpg)  
  反対側のモータも同様に取り付け  
  ![](images/assembly_motor3.jpg)

1.  天板に測域センサをネジ止め  
  天板の表側に、写真の向きにURGを載せる  
  ![](images/assembly_urg1.jpg)  
  裏面からM3のネジ2本で固定  
  ![](images/assembly_urg2.jpg)

1.  天板にキャスターをネジ止め  
  天板の表側からM4のネジを挿入し  
  ![](images/assembly_caster1.jpg)  
  裏面からM4のナットで固定  
  ![](images/assembly_caster2.jpg)

1.  モータにホイールをイモネジ止め  
  モータ軸のDカット(平らな面)に、ホイールのイモネジが入っているねじ穴を合わせ、六角レンチでイモネジを締めて固定  
  ![](images/assembly_wheel.jpg)

1.  天板にモータドライバをネジ止め  
  天板の裏面に、写真の向きでモータドライバを載せ  
  ![](images/assembly_driver1.jpg)  
  表面からM3のネジで固定  
  ![](images/assembly_driver2.jpg)

1.  天板に電池ボックスを貼り付け  
  天板の裏面に、写真の向きで、両面テープを用いて電池ボックスを固定  
  ![](images/assembly_battery.jpg)  
  貼り合わせ後、よく押しつけて、_接着面同士を十分に密着_{:style="color: red"}させます。_接着力を発揮させるため、貼り付け後5分間は、電池ボックスが上側にある状態を維持して下さい。_{:style="color: red"}

1.  配線  
  モータ・電池ボックスのケーブルをモータドライバに接続  
  ![](images/assembly_cable.jpg)  

以上でロボットの組み立ては完了です。  
![](images/robot.jpg)

# 昨年以前のロボットのメンテナンス

2014年、2015年、2016年のロボットを使用する場合、ネジ類が緩んでいないか確認しましょう。特に、ホイールをモータに固定しているイモネジの緩みに注意して下さい。

# 初期設定

1.  ノートPCでUbuntuを起動

1.  USBデバイスを開けるように、使用しているユーザをdialoutグループに追加
```shell
$ sudo adduser [ YOUR_USER_NAME ] dialout
```

1.  上記設定を反映するため、画面右上のアイコンから、ログアウトを選択してログアウト

1.  ログイン画面から再度ログイン

# 制御ソフトのインストールと動作テスト

1.  画面左のランチャーから「端末」を起動し、下記コマンドを実行して制御ソフトをインストール（日本語環境の場合は`Downloads`を`ダウンロード`に読み替えてください）
```shell
$ cd ~/Downloads/
$ git clone https://github.com/openspur/yp-spur.git
$ cd yp-spur
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
$ sudo ldconfig
```  
  ロボットパラメータファイルをダウンロードします。([参考資料: ロボットの見分け方](robot_models))
  - 2015年度・2016年度・2018年度のロボットの場合
```shell
$ mkdir ~/params
$ cd ~/params/
$ wget https://at-wat.github.io/ROS-quick-start-up/files/rsj-seminar2016.param
```  
  - 2014年度のロボットの場合
```shell  
$ mkdir ~/params
$ cd ~/params/
$ wget https://at-wat.github.io/ROS-quick-start-up/files/rsj-seminar2014.param
```
1.  ロボットの電池ボックスに電池を挿入

1.  電池ボックスの側面にあるスライドスイッチをON

1.  モータドライバのUSBをPCに接続

1.  端末を開いて、下記コマンドで制御ソフトを起動
```shell
$ ypspur-coordinator -p ~/params/rsj-seminar20??.param【該当するものに置き換えること】\
    -d /dev/serial/by-id/usb-T-frog_project_T-frog_Driver-if00
```

1.  ホイールを持ち上げて走り出さない状態、もしくは、ひっくり返してホイールが浮いている状態にしてサンプルプログラムを起動 (端末をもう1つ開いて実行します。)
```shell
$ cd ~/Downloads/yp-spur/build
$ ./samples/run-test
```  
  ![](pic/run-test.png)  
  ホイールが回転して、回転方向を何度か変え、最終的に止まることを確認したら、__Ctrl+c__{:style="border: 1px solid black"}で停止させます。このとき、ypspur-coordinatorは、まだ停止させないで下さい。(停止させた場合はもう一度起動)

1.  床の広い場所でサンプルプログラムを起動
```shell
$ ./samples/run-test
```  
  1m x 0.1m の四角形を描いてロボットが移動することを確認します。最後に、サンプルプログラムと、ypspur-coordinatorを  __Ctrl+c__{:style="border: 1px solid black"}  で停止させます。

1.  電池ボックスのスライドスイッチをOFF (待機電力で電池の電力を消費してしまいます)

## 補足
今回のセミナではYP-Spurをソースコードからインストールしましたが、
下記コマンドでバイナリーのみをインストールすることも可能です。
```shell
$ sudo apt-get install ros-kinetic-ypspur
```
