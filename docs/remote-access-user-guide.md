# AWS SSM接続の利用者ガイド

## 1. このガイドについて

本書は、AWS利用許可のある研究室メンバーが、学外・自宅などからQuadra-6000経由でGPUサーバーへ接続する手順である。学内でもAWSへの必要な通信が可能なら使用できる。

[システム全体と接続経路の比較](system-overview.md)に、`-campus` との違い、SSOとSSHの役割、初回に受け取る設定情報をまとめている。

研究室外の学内利用者・ゼミ生は、AWSを使わない[学内利用者マニュアル](campus-user-guide.md)を使用する。研究室内の利用者は`-lan`・`-campus`・`-ssm`を利用でき、本書はそのうち`-ssm`の手順を扱う。

アカウント発行、権限変更、共有領域、利用終了は管理者へ連絡する。

問い合わせ先の管理者は、中沢実（[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)）である。

```text
利用者PC
  ├── IAM Identity Center + MFA
  └── SSH秘密鍵
          │
          ▼
AWS Systems Manager → Quadra-6000 → GPUサーバー
```

AWS認証はQuadraまでのSSM経路を許可し、FreeIPA userとSSH公開鍵はQuadraおよびGPUサーバーへのloginを許可する。両方が必要である。SSOログインだけではGPUへログインできず、SSH configの `User` にはAWSユーザー名ではなくFreeIPAユーザー名を指定する。`quadra-ssm` の `ProxyCommand` がSSMの通信路を作り、GPUの `ProxyJump` がQuadraを中継する。学内踏み台のアカウントやcampusトンネルは使わない。

現在のGPU接続対象はRTX-A6000x2、AIsawa（RTX 6000 Ada搭載機）、RTX-6000ada、RTX4090、RTX5090の5台である。AIsawaの正式FQDNは`aisawaminori.i2lab.test`だが、接続時は短いSSH alias `aisawa-ssm`を使用する。利用者は申請・承認されたGPUだけへ接続する。

## 2. 守ること

- AWS access key、MFA device、SSH秘密鍵、秘密鍵passphraseを共有・送信しない
- SSH秘密鍵をサーバー、Git、クラウド同期folder、チャットへコピーしない
- NFS、LDAP、Kerberos、FreeIPA Web UI、SSH portをインターネットへ公開しない
- 利用を終える場合、端末を紛失した場合、鍵漏えいが疑われる場合は直ちに管理者へ連絡する

公開鍵（`.pub`）とfingerprintは管理者へ提出してよいが、秘密鍵は提出しない。

## 3. 初回設定

### 3.1 管理者へ申請する

次を管理者（中沢実、[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)）へ送る。

通常は、[リモートGPUアクセス申請フォーム](https://forms.cloud.microsoft/r/mgFXy680J1)へ入力して送信する。個別事情がある場合だけ、管理者へemailで相談する。

| 項目 | 送る内容 |
| --- | --- |
| 氏名 | 本人確認に使う氏名 |
| 希望するFreeIPA username | 英小文字・数字・`-`を使用し、他利用者と重複しない名前 |
| 大学または研究室のemail address | 連絡・利用終了確認に使うaddress |
| SSH公開鍵 | `.pub` fileの**1行全体**（`ssh-ed25519 AAAA...`からcommentまで） |
| SSH公開鍵fingerprint | `ssh-keygen -lf ... -E sha256`で表示される`SHA256:...` |
| 利用するGPUサーバー | 必要なhost名と用途 |
| 利用する共有領域 | 通常は`labusers`。特別project領域が必要な場合はproject名 |
| 利用期間 | 開始日と終了予定日 |

公開鍵とfingerprintは**両方**を送る。公開鍵はFreeIPAへ登録するために必要であり、fingerprintは管理者が登録した鍵と利用者の鍵が一致することを、短い文字列で相互確認するために必要である。秘密鍵、秘密鍵passphrase、AWS credential、MFA codeは送らない。

鍵が未作成なら、[SSH鍵の作成とFreeIPA登録](ssh-key-setup.md)を先に進める。アカウント発行前でも鍵は作成できる。希望名を鍵のコメントに書くだけでは登録されないため、公開鍵登録完了と確定ユーザー名の通知を待って接続する。

### 3.2 利用するOS別の初期設定手順

利用するPCのOSに対応する手順を最初から最後まで実施する。各手順には、必要software、SSH鍵の作成と提出、AWS IAM Identity Center、SSH config、VS Codeの設定、接続確認を含む。

| 利用者PC | 初期設定手順 |
| --- | --- |
| macOS | [macOS向け利用手順](remote-access-macos-guide.md) |
| Windows 11 | [Windows 11向け利用手順](remote-access-windows-guide.md) |
| Ubuntu 22.04／24.04 | [Ubuntu向け利用手順](remote-access-ubuntu-guide.md) |

管理者へ送るのは、SSH公開鍵の1行全体と`SHA256:...` fingerprintだけである。秘密鍵、passphrase、MFA code、AWS credentialは送らない。

## 4. 日常の接続

### 4.1 AWSへloginする

```bash
aws sso login --profile i2lab
```

接続先が利用できない場合は、発生時刻とエラーを管理者へ連絡する。

### 4.2 Terminalから接続する

```bash
ssh quadra-ssm
ssh rtx-a6000x2-ssm
ssh aisawa-ssm
ssh rtx-6000ada-ssm
ssh rtx4090-ssm
ssh rtx5090-ssm
```

RTX5090（`192.168.73.163`、`rtx5090.i2lab.test`）は`rtx5090-ssm`を使用する。既存利用者は[SSH設定例](../config/ssh_config.example)の対応Host blockを端末のSSH configへ追加し、placeholderを本人の設定へ置き換える。

接続後は次を確認する。

```bash
hostname -f
id
printf '%s\n' "$SHELL"
```

GPUサーバーへのSSHは、利用者PCからGPUまで暗号化される。秘密鍵をQuadraへコピーする必要はない。

### 4.3 NFS共有を使う

Quadra-6000はNFS Serverであり、GPUサーバーはNFS Clientである。接続先によってpathが異なる。

ゼミ生の標準共有領域は`labusers`である。`go2poc`は、管理者がmemberへ明示的に追加した利用者だけが使う特別なproject共有であり、通常のゼミ利用では使用しない。

| 接続先 | 標準共有（`labusers`） | 特別project共有（`go2poc`） |
| --- | --- | --- |
| Quadra-6000 | `/srv/nfs/shared/labusers` | `/srv/nfs/shared/go2poc` |
| GPUサーバー | `/mnt/nfs-shared/labusers` | `/mnt/nfs-shared/go2poc` |

親directoryの一覧表示は制限されているため、`ls /mnt/nfs-shared`は拒否される場合がある。利用を許可されたdirectoryを直接指定する。

ゼミ生の通常利用では次を使う。

```bash
cd /mnt/nfs-shared/labusers
pwd
```

`go2poc`を管理者から許可された場合だけ、`/mnt/nfs-shared/go2poc`を使用する。project共有で作成した新規fileは`go2poc` groupへ継承される。対象groupやdirectoryへ入れない場合は管理者へ連絡する。

### 4.4 GPUサーバーでDockerを使う

RTX-A6000x2、AIsawa（RTX 6000 Ada搭載機）、RTX-6000ada、RTX4090、RTX5090では、`labusers`または`go2poc`へ追加された利用者はsudoなしでDockerを実行できる。

RTX5090も共通ACL設定によるsudoなしのDocker利用に対応する。

```bash
docker version
docker ps
```

Docker実行権限はroot相当である。他利用者のcontainer、image、volume、networkを停止・削除・変更しない。containerを起動する前に、GPU、port、volume、利用期間を関係者と確認する。

### 4.5 VS Code Remote-SSHから接続する

1. VS CodeへRemote - SSH extensionを導入する
2. `Remote-SSH: Connect to Host...`を開く
3. `rtx-a6000x2-ssm`、`rtx4090-ssm`、`rtx5090-ssm`など、利用を許可された対象hostを選ぶ
4. 初回だけplatformとしてLinuxを選ぶ
5. Terminalで`hostname -f`、`id`、`echo $SHELL`を確認する

SSO sessionが期限切れの場合は、VS Codeで再試行する前にTerminalで`aws sso login --profile i2lab`を実行する。shell設定を変更した直後はRemote Windowを閉じて再接続する。

## 5. 問題が起きたとき

| 症状 | まず行うこと |
| --- | --- |
| `Unable to locate credentials` | `aws sso login --profile i2lab` |
| `AccessDeniedException` | profile名と管理者からのassignmentを確認する |
| `TargetNotConnected` | 数分待って再試行し、継続する場合は管理者へ連絡する |
| `session-manager-plugin not found` | Pluginを導入し、PATHを確認する |
| `Permission denied (publickey)` | username、鍵path、passphraseを確認する |
| Quadraへ接続できるがGPUへ接続できない | 正しい`-ssm`接続名を使う |
| VS Codeだけ接続できない | 先にTerminalで`ssh HOST`を試す |

管理者へはerror message、発生時刻、接続名、`aws sts get-caller-identity`のARNを送る。AWS secret、MFA code、password、秘密鍵、秘密鍵passphraseは送らない。
