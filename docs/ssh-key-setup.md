# 利用者のSSH鍵の作成とFreeIPAアカウントへの登録

**自分のPCで秘密鍵と公開鍵のペアを作り、公開鍵だけを希望するFreeIPAユーザー名と一緒に申請します。** 管理者が確定したアカウントへ公開鍵を登録した後、そのユーザー名と手元の秘密鍵をSSH設定に指定して接続します。`-lan`・`-campus`・`-ssm`で鍵の作り方は共通です。

## 1. 希望アカウントとSSH鍵の関係

FreeIPAは、Quadra・GPUで使う利用者名、グループ、公開鍵をまとめて管理する仕組みです。利用者がFreeIPA上で秘密鍵を作る操作はありません。鍵はアカウントの発行前にも作成できます。

例えば希望名が `taro` の場合、次の順序で利用を始めます。

1. 自分のPCで鍵を作る。秘密鍵はPCに保管する。
2. 希望名 `taro`、公開鍵の1行全体、公開鍵の指紋を申請する。
3. 管理者が名前を確定し、そのFreeIPAアカウントに公開鍵と必要な利用権限を登録する。
4. 通知された確定名をSSH configの `User`、秘密鍵の保存先を `IdentityFile` に指定する。
5. 許可されたサーバーへ接続し、`whoami` が確定名と一致することを確認する。

PCのログイン名とFreeIPAユーザー名は異なっていて構いません。申請した希望名が変更された場合も、**同じ鍵を使えます**。SSH設定には管理者から通知された確定名を使ってください。

| 項目 | 意味・扱い |
| --- | --- |
| FreeIPAユーザー名 | Quadra・GPUへログインする名前。管理者が確定する |
| 秘密鍵 `id_ed25519_i2lab` | 本人のPCで認証に使うファイル。提出しない |
| 公開鍵 `id_ed25519_i2lab.pub` | 管理者が本人のアカウントへ登録するファイル |
| 鍵のコメント（例：`taro@i2lab.test`） | 鍵を見分けるメモ。アカウントの作成・指定・登録は行わない。メールアドレスである必要もない |
| 鍵のパスフレーズ | PC上の秘密鍵を保護するために自分で決める文字列。FreeIPAやAWSのパスワードとは別 |
| 公開鍵の指紋 `SHA256:...` | 提出した鍵と登録された鍵を照合するための短い識別値。指紋だけでは公開鍵の登録はできない |

`ssh-keygen` の `-C` はコメントの指定です。`-C "taro@i2lab.test"` と書くだけで `taro` のアカウントに結び付くわけではありません。鍵の保存名にもユーザー名を含める必要はありません。オプションの意味は[OpenSSH公式マニュアル](https://man.openbsd.org/ssh-keygen)でも確認できます。

## 2. 作業するPCと既存の鍵を確認する

これ以降のコマンドはすべて**普段接続に使う自分のPC**で実行します。サーバーへSSHした端末では実行しません。Windowsは普段利用する本人のアカウントのPowerShellを開きます。鍵の作成に管理者権限は不要です。

macOSはターミナル、WindowsはPowerShell、Ubuntuは端末で確認します。

```text
ssh -V
```

バージョンが表示されれば進めます。Windowsで見つからない場合は「オプション機能」からOpenSSHクライアントを追加します。Ubuntuは `sudo apt update` と `sudo apt install openssh-client` で導入します。

同じ保存名のファイルがないか確認します。これはファイル名の確認で、秘密鍵の内容は表示しません。

**macOS・Ubuntu：**

```bash
ls -l ~/.ssh/id_ed25519_i2lab ~/.ssh/id_ed25519_i2lab.pub
```

**Windows PowerShell：**

```powershell
Test-Path "$env:USERPROFILE\.ssh\id_ed25519_i2lab"
Test-Path "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub"
```

両方が「存在しない」（macOS・Ubuntuでは `No such file or directory`、Windowsでは `False`）なら次へ進みます。片方でも存在する場合は作成コマンドを実行せず、既存の本人の鍵を使うか、別の保存名にするか確認してください。別名にした場合は以降のコマンドと `IdentityFile` もその名前に合わせます。上書きを聞かれた場合は必ず `n` で中止します。

## 3. 鍵を作成する

以下の `USERNAME` だけを希望するFreeIPAユーザー名（発行済みなら確定名）に置き換えます。これは識別用コメントです。例：`-C "taro@i2lab.test"`。

**macOS・Ubuntu：**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_i2lab -C "USERNAME@i2lab.test"
```

**Windows PowerShell：**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.ssh" | Out-Null
ssh-keygen -t ed25519 -a 100 -f "$env:USERPROFILE\.ssh\id_ed25519_i2lab" -C "USERNAME@i2lab.test"
```

実行すると次の入力を求められます。

```text
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

1行目で本人だけが管理する、推測されにくいパスフレーズを入力し、2行目で同じものを再入力します。**空にはしません。入力中は文字や「*」が表示されなくても正常です。** パスフレーズをコマンドに書き込まないでください。

`Your identification has been saved in ...` と `Your public key has been saved in ...pub` が表示されれば作成完了です。`-t ed25519` は鍵の方式、`-a 100` は保存時に秘密鍵をパスフレーズで保護する処理の反復回数、`-f` は秘密鍵の保存先です。

macOS・Ubuntuでは作成後に秘密鍵の権限を確認・設定します。

```bash
chmod 600 ~/.ssh/id_ed25519_i2lab
```

Windowsでは本人のプロファイル内に保管し、他人と共有するフォルダーへ移動しないでください。

## 4. 公開鍵と指紋を提出する

**macOS・Ubuntu：**

```bash
cat ~/.ssh/id_ed25519_i2lab.pub
ssh-keygen -lf ~/.ssh/id_ed25519_i2lab.pub -E sha256
```

**Windows PowerShell：**

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub"
ssh-keygen -lf "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub" -E sha256
```

最初のコマンドで表示される `ssh-ed25519 AAAA... コメント` の**1行全体**を提出します。画面上で折り返されていても、途中に改行や空白を挿入しません。2つ目の出力にある `SHA256:...` が指紋です。`AAAA...` や `SHA256:...` は説明用の省略表記であり、実際の申請には自分の出力を省略せず使用します。

申請フォームまたは管理者指定の連絡先へ、次を提出します。

- 希望するFreeIPAユーザー名（発行済みなら確定名）
- 公開鍵 `.pub` の1行全体とその指紋
- 利用する経路（`-lan`・`-campus`・`-ssm`）、希望GPU、その他の申請項目

拡張子なしのファイルは秘密鍵です。`-----BEGIN OPENSSH PRIVATE KEY-----` と表示された場合は秘密鍵を開いています。その内容を提出しないでください。秘密鍵・パスフレーズ・パスワードは、サーバー、共有領域、Git、Issue、チャットへコピーしません。

`-campus` では、管理者が**踏み台の中継専用 `campus-relay` と本人のFreeIPAアカウントの両方**へ公開鍵を登録します。同じ本人の鍵を使う設定例ですが、踏み台用に別鍵を作るよう指示された場合は、本人のPCで別名の鍵を作り、その公開鍵を提出してください。管理者がトンネル維持に使う専用鍵は利用者の鍵とは別です。

## 5. 登録完了後にSSH設定へ反映する

管理者から確定したFreeIPAユーザー名、公開鍵登録完了と指紋、許可GPU、サーバーのホスト鍵指紋を受け取ります。`-campus` では踏み台の `campus-relay` への本人の公開鍵登録完了と中継利用許可も必要です。**鍵を作っただけ、または申請しただけでは接続できません。**

利用する経路の設定例を自分のPCのSSH configへ取り込みます。`-campus` の `campus-jump` は `User campus-relay` を使用します。Quadra・GPUの `User` には本人のFreeIPAユーザー名を設定します。

- `-lan`・`-ssm`：[直接接続・AWS経由の設定例](../config/ssh_config.example)
- `-campus`：[学内踏み台経由の設定例](../config/ssh_config.campus.example)

| 設定例の置換箇所 | 設定する値 |
| --- | --- |
| `__SSH_USER__` | 管理者から通知されたFreeIPAユーザー名。例：`taro` |
| `__IDENTITY_FILE__` | 秘密鍵のパス。macOS・Ubuntuの例：`~/.ssh/id_ed25519_i2lab`。`.pub` は付けない |

Windowsの秘密鍵パスは `"C:/Users/本人のWindowsユーザー名/.ssh/id_ed25519_i2lab"` のように指定します。このパスのユーザー名は**PCのユーザー名**で、`User` に書くFreeIPAユーザー名とは区別します。

例えば `-campus` でRTX5090が許可されている場合は、次で読み込まれた設定を確認します。

```bash
ssh -G rtx5090-campus
```

`user` が確定名、`identityfile` が作成した秘密鍵、`proxyjump` が `quadra-campus` になっているか確認します。`ssh -G` は設定表示だけで、登録・到達性・接続成功は確認しません。その後は[学内接続](campus-user-guide.md#6-順番に接続を確認する)、[直接接続](lan-user-guide.md)、[SSM接続](remote-access-user-guide.md)の該当手順へ進みます。

## 6. よくある疑問・入力要求

| 状況 | 意味と対応 |
| --- | --- |
| 希望名と確定名が違う／コメントにPCのユーザー名が入っている | 鍵の作り直しは不要。公開鍵の登録先とSSH設定の `User` を確定名に合わせる |
| `Enter passphrase for key ...` | PC上の秘密鍵を開くための入力。作成時のパスフレーズを入力する |
| `user@host's password:` | サーバーのアカウントのパスワード要求。鍵のパスフレーズを入力する場所ではない。接続先・`User`・鍵の指定と登録状況を確認する |
| 初回接続時の `SHA256:...` | サーバーのホスト鍵指紋。提出した自分の公開鍵指紋とは別。管理者から通知された接続先の指紋と照合する |
| `Permission denied (publickey)` | 確定ユーザー名、`IdentityFile`、登録された公開鍵の指紋を確認し、拒否された接続先とエラーを管理者へ連絡する |
| パスフレーズを忘れた | 復元できないため別名で新しい鍵を作り、公開鍵の差し替えを管理者へ依頼する |
| PCを追加・交換する | 新しいPCで別の鍵を作り、公開鍵の追加・差し替えを申請する |
| PCを紛失した／秘密鍵が漏れた疑いがある | 直ちに管理者へ連絡し、旧公開鍵の登録解除と新しい鍵への更新を依頼する。`-campus` は踏み台・FreeIPA両方が対象 |

[利用マニュアルの入口へ戻る](README.md)
