# 利用者のSSH鍵の準備

## 3. 自分のPCでSSH鍵を準備する

### 3.1 SSHコマンドの確認

macOSはターミナル、WindowsはPowerShell、Ubuntuは端末を開きます。

```text
ssh -V
```

バージョンが表示されれば進めます。コマンドがない場合、Windowsは「オプション機能」からOpenSSHクライアントを追加します。Ubuntuは以下で導入します。

```bash
sudo apt update
sudo apt install openssh-client
```

### 3.2 鍵を作成する

既に同名の鍵を使っている場合は上書きしません。鍵作成時に上書きを聞かれたら `n` で中止し、既存鍵の利用について管理者へ確認してください。利用者の鍵にはパスフレーズを設定します。入力中は文字が表示されません。

**macOS・Ubuntu：**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_i2lab
cat ~/.ssh/id_ed25519_i2lab.pub
ssh-keygen -lf ~/.ssh/id_ed25519_i2lab.pub -E sha256
```

**Windows PowerShell：**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.ssh" | Out-Null
ssh-keygen -t ed25519 -a 100 -f "$env:USERPROFILE\.ssh\id_ed25519_i2lab"
Get-Content "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub"
ssh-keygen -lf "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub" -E sha256
```

`ssh-ed25519 AAAA...` から末尾までが公開鍵です。改行を挿入せず1行全体を提出します。拡張子なしの `id_ed25519_i2lab` は秘密鍵で、自分のPCに保管します。サーバーや共有フォルダ、Git、チャットへコピーしません。


準備後は[利用マニュアルの入口](README.md)へ戻り、該当する接続手順を進めてください。
