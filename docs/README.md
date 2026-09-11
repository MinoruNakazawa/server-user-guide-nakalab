# GPUサーバー利用マニュアル

自分の利用区分を選んでください。初めて利用する場合は、申請・承認と本人のSSH公開鍵の登録が必要です。

- [システム全体と接続経路の選び方](system-overview.md)：構成図、`-campus` と `-ssm` の違い、アカウントと共有領域の仕組み
- [SSH鍵の作成とFreeIPAアカウントへの登録](ssh-key-setup.md)：希望ユーザー名との関係、OS別の鍵作成、公開鍵・指紋の提出、SSH設定への反映

| 利用区分 | 最初に読むところ | 接続方法 | AWSの準備 |
| --- | --- | --- | --- |
| 研究室内の利用者 | [研究室内利用者の入口](#lab-users) | 直接接続の`-lan`、AWS経由の`-ssm` | `-ssm`の場合に必要 |
| 研究室外の学内利用者・ゼミ生 | [研究室外利用者の入口](#campus-users) | 学内踏み台経由の`-campus` | 不要 |

研究室内の利用者も、学内踏み台へ到達できる環境では`-campus`を利用できます。研究室外の利用者は本書では**学内からの利用者**を指し、AWS利用権限は発行しません。

<a id="lab-users"></a>

## 研究室内利用者：`-lan`・`-ssm`

研究室LANなどからGPUのSSHへ直接到達できる場合は`-lan`、自宅・学外などからAWS経由で接続する場合は`-ssm`を使います。

| 手順 | 参照先 |
| --- | --- |
| 1. 利用を申請する | [利用申請フォーム](https://forms.cloud.microsoft/r/mgFXy680J1)。利用目的・期間・対象GPU・必要な接続経路を申請し、[SSH鍵の準備](ssh-key-setup.md)で作成した本人の公開鍵と指紋を提出 |
| 2. 承認とアカウント通知を受け取る | FreeIPAユーザー名、公開鍵登録完了、許可GPUを確認。`-ssm`を使う人はAWSの設定情報も受領 |
| 3. SSH configを設定する | [研究室内向け設定例](../config/ssh_config.example)の冒頭コメントに従う |
| 4. `-lan`で接続する | [直接接続の利用手順](lan-user-guide.md)。AWS SSO loginは不要 |
| 5. `-ssm`で接続する | [SSM利用者ガイド](remote-access-user-guide.md)と下記のOS別初期設定を使用 |

**AWS SSMを利用する場合のOS別初期設定：**

- [macOS](remote-access-macos-guide.md)
- [Windows](remote-access-windows-guide.md)
- [Ubuntu](remote-access-ubuntu-guide.md)

これらのOS別手順はAWS設定を含みます。`-lan`だけを使う場合、AWS関連の導入・認証は不要です。

設定後は、許可された接続先でホスト名・本人のユーザー名・GPU認識を確認してください。RTX5090を使用する例です。

```bash
# 直接接続できる場所から
ssh rtx5090-lan 'hostname -f; whoami; nvidia-smi'

# AWS SSMを使用する場合（本人が設定したprofile名に合わせる）
aws sso login --profile i2lab
ssh rtx5090-ssm 'hostname -f; whoami; nvidia-smi'
```

全GPUの接続名は[研究室内向け設定例](../config/ssh_config.example)のHost行を参照してください。研究室内から`-campus`を使う場合は、下記の学内利用者マニュアルのSSH設定・動作確認を使います。

<a id="campus-users"></a>

## 研究室外の学内利用者：`-campus`

**[学内利用者マニュアル：申請から動作確認まで](campus-user-guide.md)を最初から順に進めてください。**

学内のPCから、踏み台の中継専用 `campus-relay` と本人のFreeIPAアカウントで接続します。踏み台でも利用者ごとに登録された本人の鍵で認証します。シェル・コマンド実行は許可されず、QuadraのSSH入口への中継だけを利用できます。

```text
自分のPC → 学内踏み台 → Quadra → 許可されたGPUサーバー
```

1. 学内利用のみであることを明記して申請する。
2. 本人のSSH鍵を準備し、公開鍵と指紋を提出する。
3. 踏み台・FreeIPA両方への登録完了と、利用許可の通知を受け取る。
4. [学内向け設定例](../config/ssh_config.campus.example)を自分のSSH configへ取り込み、ユーザー名・鍵パスを置き換える。
5. マニュアルの確認表に沿って、Quadra・GPU・共有領域・必要に応じてVS Codeの動作を確認する。

設定後の接続例：

```bash
ssh rtx5090-campus 'hostname -f; whoami; nvidia-smi'
```

利用者がトンネルを起動・常駐化する必要はありません。踏み台へ到達できない、またはトンネルが停止している場合は管理者へ連絡してください。

## 問い合わせ

接続・利用許可に関する問い合わせは、中沢実（[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)）へ連絡してください。
