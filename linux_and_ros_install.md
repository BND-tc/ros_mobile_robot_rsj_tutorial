---
title: Ubuntu LinuxとROSのインストール
date: 2018-05-11
---

- Table of contents
{:toc}

# Ubuntu LinuxとROSのインストール

セミナー中に開発環境としてUbuntu Linuxを利用します。そして、Ubuntu
Linux上でROSを利用します。本ページではUbuntu
LinuxとROSのインストール方法を示します。

# 用意するもの

- ノート型パソコン
-- *本手順によりパソコンの既存のOS（Windows等）及び保存されているデータやソフトウェアは完全にが削除されます。予めバックアップを行ってください。*{: style="color: red"}
- 容量４GB以上の空のUSBメモリ
- インターネット接続

# 手順

## Ubuntu Linuxのダウンロード

1. 下記URLからUbuntu Linux 16.04(64bit)のインストールイメージをダウンロードします。

   [Ubuntu ダウンロード](http://releases.ubuntu.com/xenial/)

   ![Ubuntu ダウンロード](/images/ubuntu_download.png)


## Live USBの作成

1. 下記URLから、Live USB作成ソフトをダウンロードします。
   - Windows、Mac OS Xの場合：<https://unetbootin.github.io/>

     ページ中の、Live USB作成に使用しているPCのOSを選択してください。

     ![UNetbootinダウンロード](/images/unetbootin_download.png)

   - Linux (Debian/Ubuntu) の場合: 下記コマンドを実行

     ```shell
     $ sudo apt-get install unetbootin
     ```

1. 事故防止のため、使用しないUSBメモリやメモリーカードを取り外し、使用するUSBメモリのみを接続します。使用するUSBメモリは、ファイルが入っていない、空の状態にして下さい。

1. Live USBを作成するPCとUbuntuをインストールするPCが別でも構いません。

1. ダウンロードした unetbootin-windows-???.exe（Windowsの場合）を実行します。

   下記、「WindowsによってPCが保護されました」画面が現れた場合は、「実行」ボタンをクリックしてください。

   ![Windows UAC](/images/windows_idiot_screen.png)

   また、下記のユーザアカウント制御画面が現れた場合、「はい」をクリックしてください。

   ![Windows UAC](/images/windows_uac.png)

1. UNetbootinの画面で「ディスクイメージ」を選択し、「…」ボタンをクリックして先ほどダウンロードした、`ubuntu-16.04.2-desktop-amd64.iso` ファイルを選択します。また、「スペースは、リブートしてもファイルを維持するために使用」欄に「4096」と入力し、「ドライブ」欄で、使用するUSBメモリのドライブ名を選択します。内容を確認後、「OK」をクリックしてください。

   ![UNetbootin process 1](/images/unetbootin_setting.png)

   完了まで、しばらく待機します。（USB2.0の場合10分以上、書き込み速度の遅いメモリだと30分程度かかる場合があります。）
   下記の「永続性を設定する」画面で、応答なしと表示される場合がありますが、正常に動作していますので、そのまま待機してください。

   ![UNetbootin process 2](/images/unetbootin_freeze.png)

   下記画面が表示されれば、「Live USB」の作成は完了です。終了をクリックして下さい。

   ![UNetbootin process 3](/images/unetbootin_end.png)

1. Live USBから起動するためのBIOSの設定を行います。

   セミナーで使用するPCの電源を切り、下記の手順で作成したLive USBを接続した状態で起動します。起動時に、BIOS設定画面に入ります。*PCのメーカー毎に入り方が異なります*{: style="color: red"}ので、マニュアル等で確認してください。図はAcerの例です。

   ![BIOS 1](/images/acer_boot.png)

   BIOS設定画面に入ったら、起動順（Boot order, Boot priority）の設定で、USBメモリが最優先になるように設定します。 （表示は使用しているPCおよびUSBメモリのメーカーによって異なります。）

   ![BIOS 2](/images/acer_boot_order1.png)

   ![BIOS 3](/images/acer_boot_order2.png)

   設定を保存して再起動します。

   ![BIOS 4](/images/acer_boot_save.png)

## Ubuntu Linuxのインストール

1. Live USBをパソコンに接続し、パソコンの電源を入れます。

1. 以下の画面が表示されます。言語を選択してください。

   ![Ubuntu install 1](/images/ubuntu_install_1.png)

1. 「Ubuntuをインストール」を選択しインストール手順を開始します。画面に出る説明に従ってインストール手順を続けてください。

   ![Ubuntu install 2](/images/ubuntu_install_2.png)

1. 以下の画面に届いたら、「ディスクを削除してUbuntuをインストールする」を選択してください。

   ![Ubuntu install 3](/images/ubuntu_install_3.png)

1. インストール後、LiveUSBを外してパソコンを再起動すると以下の画面が現れます。これでUbuntu Linuxのインストールが完了です。

   ![Ubuntu install 4](/images/ubuntu_install_4.png)


## ROSのインストール

1. *ROS Kinetic Kame*{: style="color: blue"}をインストールします。

   ROSをインストールするために、以下のURLで書いてある手順に従ってください。すべてのデスクトップ環境のインストールを行ってください。

   [ROS KineticのUbuntuへのインストール](http://wiki.ros.org/ja/kinetic/Installation/Ubuntu)

   以下はインストールコマンドの概要だけです。上記のページをご参照ください。

   ```shell
   $ sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) \
       main" > /etc/apt/sources.list.d/ros-latest.list'
   $ sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 \
       --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
   $ sudo apt-get update
   $ sudo apt-get install ros-kinetic-desktop-full
   $ sudo rosdep init
   $ rosdep update
   $ echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
   $ source ~/.bashrc
   $ sudo apt-get install python-rosinstall
   ```

1. 確認のため、新しい端末を起動して下記を実行してください。

   ```shell
   $ printenv | grep ROS
   ```

   下記が出力されたら、ROSのインストールが完了しました。

   ```shell
   ROS_ROOT=/opt/ros/kinetic/share/ros
   ROS_PACKAGE_PATH=/opt/ros/kinetic/share
   ROS_MASTER_URI=http://localhost:11311
   ROSLISP_PACKAGE_DIRECTORIES=
   ROS_DISTRO=kinetic
   ROS_ETC_DIR=/opt/ros/kinetic/etc/ros
   ```

## 必要なパッケージのインストール

最後に、本セミナーに必要なパッケージをインストールします。以下のコマンドを実行しインストールしてください。

```shell
$ sudo apt update
$ sudo apt install ros-kinetic-slam-gmapping ros-kinetic-mouse-teleop
$ sudo apt install ros-kinetic-openni2-camera
$ sudo apt install ros-kinetic-openni2-launch
$ sudo apt install ros-kinetic-urg-node
$ sudo apt install ros-kinetic-hokuyo3d
$ sudo apt install ros-kinetic-pointcloud-to-laserscan
$ sudo apt install ros-kinetic-map-server ros-kinetic-move-base
$ sudo apt install ros-kinetic-amcl ros-kinetic-dwa-local-planner
$ sudo apt install ros-kinetic-navigation
```
## ModemManagerを削除 

```shell
sudo apt-get purge modemmanager
```
これは、ロボット・センサのUSBが接続されたときに、ModemManagerが起動していると1～2分間ほどモデムとしての通信試行処理が実行され、その間デバイスが利用できない問題があるためです。 そのため、削除、起動させない、または、USBデバイスをModemManagerのblacklistに追加する、といった処置が必要になります。

以上、開発環境の構築が完了しました。
