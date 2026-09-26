# GPUサーバー利用マニュアル

自分の利用区分を選んでください。初めて利用する場合は、申請・承認と本人のSSH公開鍵の登録が必要です。

- [システム全体と接続経路の選び方](system-overview.md)：構成図、`-lan` と `-campus` の違い、アカウントと共有領域の仕組み
- [SSH鍵の作成とFreeIPAアカウントへの登録](ssh-key-setup.md)：希望ユーザー名との関係、OS別の鍵作成、公開鍵・FingerPrint（指紋）の提出、SSH設定への反映

| 利用区分 | 最初に読むところ | 接続方法 |
| --- | --- | --- |
| 研究室内の利用者 | [研究室内利用者の入口](#lab-users) | 直接接続の`-lan`。到達性に応じて`-campus` |
| 研究室外の学内利用者・ゼミ生 | [研究室外利用者の入口](#campus-users) | 学内踏み台経由の`-campus` |

研究室内の利用者も、学内踏み台へ到達できる環境では`-campus`を利用できます。学外からは[remote-VPN利用案内](https://uranus.mars.kanazawa-it.ac.jp/dpc/navi_network/navi_network_5/)に従ってVPNへ接続し、VPN経由で到達できる接続名を使います。`-ssm` は使用しません。

<a id="lab-users"></a>

## 研究室内利用者：`-lan`・`-campus`

GPUのSSHへ直接到達できる場合は`-lan`を使います。学外・自宅では先にremote-VPNへ接続し、GPUへ直接届けば`-lan`、学内踏み台へ届けば`-campus`を使います。

| 手順 | 参照先 |
| --- | --- |
| 1. 利用を申請する | [利用申請フォーム](https://forms.cloud.microsoft/r/mgFXy680J1)。利用目的・期間・対象GPU・必要な接続経路を申請し、[SSH鍵の準備](ssh-key-setup.md)で作成した本人の公開鍵とFingerPrint（指紋）を提出 |
| 2. 承認とアカウント通知を受け取る | FreeIPAユーザー名、公開鍵登録完了、許可GPUを確認 |
| 3. SSH configを設定する | `-lan` は[直接接続の設定例](../config/ssh_config.example)、`-campus` は[学内踏み台経由の設定例](../config/ssh_config.campus.example)に従う |
| 4. `-lan`で接続する | [直接接続の利用手順](lan-user-guide.md) |
| 5. `-campus`で接続する | [学内踏み台経由の利用手順](campus-user-guide.md) |

学外から利用する場合は、大学の案内に従ってremote-VPNを接続してからSSHの到達性を確認します。VPNへの接続だけではGPU利用権限は付与されません。

設定後は、許可された接続先でホスト名・本人のユーザー名・GPU認識を確認してください。RTX5090を使用する例です。

```bash
# 直接接続できる場所から
ssh rtx5090-lan 'hostname -f; whoami; nvidia-smi'

# VPN接続後、学内踏み台へ到達できる場合
ssh rtx5090-campus 'hostname -f; whoami; nvidia-smi'
```

全GPUの接続名は[直接接続の設定例](../config/ssh_config.example)と[学内踏み台経由の設定例](../config/ssh_config.campus.example)のHost行を参照してください。研究室内から`-campus`を使う場合は、下記の学内利用者マニュアルのSSH設定・動作確認を使います。

<a id="campus-users"></a>

## 研究室外の学内利用者：`-campus`

**[学内利用者マニュアル：申請から動作確認まで](campus-user-guide.md)を最初から順に進めてください。**

学内のPCから、踏み台の中継専用 `campus-relay` と本人のFreeIPAアカウントで接続します。踏み台でも利用者ごとに登録された本人の鍵で認証します。シェル・コマンド実行は許可されず、QuadraのSSH入口への中継だけを利用できます。

```text
自分のPC → 学内踏み台 → Quadra → 許可されたGPUサーバー
```

1. 学内利用のみであることを明記して申請する。
2. 本人のSSH鍵を準備し、公開鍵とFingerPrint（指紋）を提出する。
3. 踏み台・FreeIPA両方への登録完了と、利用許可の通知を受け取る。
4. [学内向け設定例](../config/ssh_config.campus.example)を自分のSSH configへ取り込み、ユーザー名・鍵パスを置き換える。
5. マニュアルの確認表に沿って、Quadra・GPU・共有領域・必要に応じてVS Codeの動作を確認する。

設定後の接続例：

```bash
ssh rtx5090-campus 'hostname -f; whoami; nvidia-smi'
```

利用者がトンネルを起動・常駐化する必要はありません。踏み台へ到達できない、またはトンネルが停止している場合は管理者へ連絡してください。

## 接続後のDocker利用（全経路共通）

[Docker利用ガイド](docker-user-guide.md)を参照してください。利用権限の確認、コンテナの準備、GPU・NFSの確認、共同利用の注意事項をまとめています。

## 問い合わせ

接続・利用許可に関する問い合わせは、中沢実（[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)）へ連絡してください。
