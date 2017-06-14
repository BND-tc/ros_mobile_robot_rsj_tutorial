---
title: 資料の修正
date: 2017-06-13
---

## 画像処理とマニピュレータの組み合わせ

1. 「ワークスペースのセットアップ」で、自分のワークスペースからのパッケージを利用する場合に、「block_finder」のパッケージ名が変更されました。

   修正前
   : ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ cp -r ~/crane_plus_ws/src/pick_and_placer .
     $ cp -r ~/block_finder_ws/src/block_finder .
     ```

   修正後
   : ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ cp -r ~/crane_plus_ws/src/pick_and_placer .
     $ cp -r ~/block_finder_ws/src/rsj_2017_block_finder .  # <- 変更
     ```

   従って、全ページで「block_finder」パッケージ名を「rsj_2017_block_finder」に変更されました。

1. 「ワークスペースのセットアップ」で、本セミナーに用意されたノードの利用する場合の「block_finder」をクローンするところでは、URLが変更されました。

   修正前
   : ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ git clone https://github.com/Suzuki1984/image_processing.git
     ```

   修正後
   : ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ git clone https://github.com/Suzuki1984/rsj_2017_block_finder  # <- 変更
     ```

1. 「ワークスペースのセットアップ」で、本セミナーに利用するカメラノードのリポジトリが移動されました。

   修正前
   : 他の必要なノードもワークスペースに入れます.

     ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ git clone https://github.com/gbiggs/crane_plus_arm.git
     $ git clone https://github.com/ros-drivers/usb_cam.git
     ```

   修正後
   : 他の必要なノードもワークスペースに入れます.

     ```shell
     $ cd ~/rsj_2017_application_ws/src
     $ git clone https://github.com/gbiggs/crane_plus_arm.git
     $ git clone https://github.com/ros-drivers/usb_cam.git  # <- 変更
     ```
