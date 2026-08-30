# ROS2スタック机上評価（担当：chat側Claude）

目的：RobOps検証の土台となる既製ROS2スタックを1本選ぶ。自作しない。

## 候補（2026-08-26調べ）
1. **legalaspro/so101-ros-physical-ai**（本命）——SO-101リーダー/フォロワー向け完全スタック。Feetech STS3215のros2_controlドライバ、テレオペ、MoveIt 2、マルチカメラ、エピソード録画（rosbag/MCAP）、LeRobot v3.0データセット変換、方策学習・推論（ACT/SmolVLA同期、policy_server非同期）、Rerun可視化。環境管理はpixi。2026-03にOpen Robotics Discourseで公開。
2. **so101_ros2ワークスペース**（so101-ros2.readthedocs.io）——bringup／controller／description／hardware_interface／teleop／データ録画ノード一式。lerobot APIブリッジあり。
3. **so_arm_100_hardware**（部品）——SO-ARM100系のros2_controlハードウェアインターフェース単体。Jazzyでdocs.ros.orgに公式ドキュメントあり。上2つが不適だった場合の組み立て用部品。

## 評価基準
1. 2カメラ録画の実績（front+wrist構成が通るか）
2. 既存較正（lerobot形式）の持ち込み可否
3. LeRobot v3変換の互換性（既存の学習パイプラインに流せるか)
4. 保守の生きの良さ（コミット頻度・Issue対応）

## 分岐表（玉1の結果待ち）
- Ubuntu 24.04 → ROS2 **Jazzy**。候補1のJazzy対応状況を確認して適合なら候補1で確定。
- Ubuntu 22.04 → ROS2 **Humble**。候補1のHumble対応状況を確認。不適なら候補2・3を精査。
- いずれの場合も、評価結果と選定理由をこのファイルに追記してから導入に進む。

## 状態
- [ ] 玉1の Ubuntu バージョン判明待ち（ENV.md）
- [ ] 候補1の詳細評価（対応distro・2カメラ・較正持ち込み）
- [ ] 選定確定と導入手順書の作成
