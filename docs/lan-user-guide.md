# 研究室内利用者：直接接続（-lan）

対象GPUのSSHへ直接到達できるネットワークで使います。AWS CLI・Session Manager Plugin・AWS SSO loginは不要です。

## 1. 申請とSSH鍵の準備

[利用申請フォーム](https://forms.cloud.microsoft/r/mgFXy680J1)で、研究室内利用者・-lan利用であることと、氏名・連絡先・利用目的・期間・希望GPUを伝えてください。本人の公開鍵とFingerPrint（指紋）も提出します。

SSH鍵がない場合は[SSH鍵の準備](ssh-key-setup.md)へ進みます。既存の鍵は上書きしません。秘密鍵とパスフレーズは提出しないでください。

## 2. 承認と設定

管理者から本人のFreeIPAユーザー名・公開鍵登録完了・許可GPU・共有領域・ホスト鍵のFingerPrint（指紋）の通知を受け取ります。
[研究室内向け設定例](../config/ssh_config.example)のコメントに従い、-lanのブロックを本人のPCへ取り込んでください。AWS用のブロックは不要です。

## 3. 動作確認

以下はRTX5090を許可された場合の例です。自分のPCで実行します。

```bash
ssh rtx5090-lan 'hostname -f; whoami; id; nvidia-smi'
```

初回は通知されたホスト鍵のFingerPrint（指紋）と照合します。`rtx5090.i2lab.test`、本人のユーザー名、承認されたグループ、GPU情報が表示されれば基本確認は成功です。nvidia-smiはGPU認識の確認であり、個別の研究プログラムの動作試験は別途行います。

共有領域は[共有領域の確認手順](remote-access-user-guide.md#43-nfs共有を使う)、Dockerは[Docker利用ガイド](docker-user-guide.md)を参照してください。この節のGPU上の操作は-lanでも共通です。SSO loginの節は実施しません。

VS Code Remote-SSHでは承認された-lan接続名を選び、リモート端末でホスト名とユーザー名を確認します。

## 4. 接続できないとき

タイムアウトする場合は接続元ネットワークと対象GPUの到達性、公開鍵認証が拒否される場合はユーザー名と鍵指定を確認します。解消しない場合は接続名・発生時刻・エラーを管理者へ連絡してください。ネットワーク条件に応じて-campusまたは-ssmを使う場合は、[利用マニュアルの入口](README.md)で必要な申請・設定を確認してください。
