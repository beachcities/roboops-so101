# QUEUE — 発注台帳

貼り先：Ubuntu機（デュアルブートのUbuntu側）の `~/roboops-so101` で起動したClaude Codeセッション。
起動手順：`git clone git@github.com:beachcities/roboops-so101.git ~/roboops-so101 && cd ~/roboops-so101 && claude`（clone済みなら `cd ~/roboops-so101 && git pull && claude`）。

## セッション共通ルール
- 実行順序は玉1→玉2→玉5→玉3。各玉は独立しており、途中の玉が失敗しても次へ進んでよい（失敗はそのまま報告する）。
- 玉4は blocked（山田同席が前提）。勝手にready化しない。
- アームを駆動するコマンドは玉1〜玉3・玉5には含まれない。含めないこと。カメラの接続・設定・ストリーミングはアーム非駆動なので承認不要（山田決裁済み 2026-08-30。理由：カメラ操作は物理的危険がなく、既存 /home/masayuki/so101/CLAUDE.md の実機コマンド要承認ルールはアーム駆動系を対象とするため）。
- 各玉の完了時に `REPORTS/YYYY-MM-DD_玉N.md` を作成してcommit・pushする。報告の冒頭定型：発注時刻は機械取得できる場合のみ記載し、取得不能なら「取得不能」と書く。開始・完了は `date` で機械取得し、所要は差から算出する。**推定禁止**。
- 触れないもの・読めなかったものは推測と明示する。
- 秘密（トークン・鍵・パスワード）をこのリポジトリにcommitしない。吸い上げ対象に混入していたら除外し、除外した旨だけ報告する。
- **このリポジトリはpublic**（2026-10-09にchatがGitHub上の公開設定で確認）。実名・メールアドレス・自宅の写った画像をcommitしない。legacy/ へのコピー時も同様に除外し、除外した旨だけ報告する。

## 玉1：環境同定（ready 2026-08-30〜／実機不要／所要目安10分）
1. `lsb_release -a`（無ければ `cat /etc/os-release`）、`uname -r`、`free -h`、`df -h /`、`lsusb -t`、`nvidia-smi`（無ければ「GPU無し」と記録）を実行。
2. 結果を `ENV.md` に整理して書く。**Ubuntuのバージョンは最重要**（24.04→ROS2 Jazzy／22.04→Humble の分岐入力になる。分岐の中身は ROS2_EVAL.md 参照）。
3. `git config user.name` と `git config user.email` が実名・実メールかどうかだけを ENV.md に「実名／非実名」「noreply／それ以外」の形で記録する（値そのものは書かない）。
4. commit・push し、REPORTS/ に報告を書く。

## 玉2：現行構成の吸い上げ（ready 2026-08-30〜／実機不要／所要目安30分）
1. `/home/masayuki/so101/` 配下の CLAUDE.md・スクリプト・設定メモ類を `legacy/so101-workspace/` へコピーする。
2. lerobot較正ファイルの所在（パス）と内容を確認し、較正ファイル本体も `legacy/calibration/` へコピーする。
3. `/home/masayuki/lerobot-venv` の `pip freeze` を `legacy/pip-freeze-lerobot-venv.txt` に保存する。
4. C920nのv4l2固定値（AF/AE/WB）が記録されているファイルを特定し、legacy/ に含める。見つからなければ `v4l2-ctl --device=<C920nのby-idパス> --list-ctrls` の現在値を保存する。
5. commit・push し、REPORTS/ に報告を書く。

## 玉5：録画データとSmolVLA対応の棚卸し（ready 2026-10-09〜／実機不要・読み取りのみ／所要目安20分）
目的：応用編1の最終課題（SO-101実機で「録画→学習→実機推論」を一周させる。締切 2026-11-02 10:00）の段1＝録画の規模を決める入力にする。
1. lerobotのローカルデータセット置き場（既定は `~/.cache/huggingface/lerobot/`、環境変数 `HF_LEROBOT_HOME` があればそちら）と `/home/masayuki/so101/` 配下を調べ、`so101-pickplace` 系（`so101-pickplace`、`so101-pickplace-test` ほか）のデータセットを全て列挙する。
2. 各データセットについて `meta/info.json` から `total_episodes`・`total_frames`・`fps`・カメラ名（features の observation.images.*）・`codebase_version` を記録し、`meta/tasks.jsonl`（または tasks.parquet）のタスク文字列を記録する。最終更新日時（`ls -l --time-style=long-iso`）も添える。
3. 動画が開けるか確認する：各データセットの最初のエピソードの動画1本について `ffprobe` でコーデック・解像度・長さを記録する（再生や画像の書き出しはしない）。
4. `/home/masayuki/lerobot-venv` の lerobot について、バージョン、`lerobot/policies/` 配下に `smolvla` と `act` があるか、`lerobot-train --help`（無ければ `python -m lerobot.scripts.train --help`）で `--policy.type` に smolvla が指定できるかを記録する。あわせて venv の python で `from lerobot.policies.smolvla.modeling_smolvla import SmolVLAPolicy` と `from lerobot.policies.act.modeling_act import ACTPolicy` を試し、成功／エラー全文を記録する（SmolVLAは追加依存 `lerobot[smolvla]`＝transformers・num2words・accelerate が要る。chatがGitHubの上流ソースで確認、2026-10-09）。インストールや更新はしない。
5. 結果を `DATASETS.md` に書く。commit・push し、REPORTS/ に報告を書く。データセット本体（動画・parquet）はcommitしない。

## 玉3：InnoMaker接続確認と2カメラ帯域実測（ready 2026-08-30〜／カメラのみ・アーム非駆動／所要目安40分）
前提：InnoMakerカメラ実物がUbuntu機の近くにあること（無ければこの玉をblockedに変更し、その旨をこのファイルに追記して報告する）。
1. InnoMakerを接続し、`ls -l /dev/v4l/by-id/` で安定パスを確認。C920nと両方のby-idパスを記録。
2. `v4l2-ctl --device=<InnoMakerのby-idパス> --list-formats-ext` で対応フォーマット（MJPEG/YUYV、解像度、fps）を記録。
3. InnoMakerのAF/露出/WBを `v4l2-ctl` で固定し、設定コマンドを記録（C920nの運用と同型に）。
4. 2カメラ同時ストリーミングの帯域実測：両カメラを640x480 MJPEG 30fpsで同時取得し、フレーム落ちの有無を確認。落ちる場合はUSBポートの分離（`lsusb -t` でコントローラ配置確認）や解像度・fpsの組を変えて、通る構成を1つ確定させる。
5. 結果一式を `CAMERA.md` に書く（by-idパス2本・固定コマンド・帯域実測の通る構成）。commit・push し、REPORTS/ に報告を書く。

## 玉4：2カメラ録画の煙試験（**blocked**：アーム駆動を伴うため山田同席が前提。玉3完了後、山田の合図でready化する）
内容（予告）：InnoMakerを養生テープでアーム先端付近に仮止めし、lerobot record で front+wrist の2視点短尺1本を録画、データセットに両視点が正しく入ることを確認する。所要目安30分。

## chat側の担当（Ubuntu機セッションの作業ではない）
- ROS2スタックの机上評価（ROS2_EVAL.md）。玉1のUbuntuバージョン判明を分岐入力として消化する。
