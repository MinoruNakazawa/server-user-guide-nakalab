# macOS向けAWS SSM接続の初期設定

本書は研究室内の`-ssm`利用者向けである。`-lan`・`-campus`のみを使う人は[利用マニュアルの入口](README.md)から該当手順へ進む。

この手順は[SSM利用者ガイド](remote-access-user-guide.md)のmacOS向け初期設定である。管理者からFreeIPA username、AWS IAM Identity CenterのStart URL・SSO region・AWS account・permission set・Quadra Managed Node IDを受け取ってから進める。

## 1. 必要softwareを導入する

Homebrewを使用する例である。未導入の場合は[Homebrew](https://brew.sh/)を導入してから実行する。

```bash
brew install awscli
brew install --cask session-manager-plugin

aws --version
session-manager-plugin --version
ssh -V
```

VS Codeを使う場合は、VS CodeとRemote - SSH extensionを導入する。

```bash
brew install --cask visual-studio-code
code --install-extension ms-vscode-remote.remote-ssh
```

## 2. SSH鍵を作成して提出する

`USERNAME`を自分のFreeIPA usernameへ置き換える。passphraseは空にせず、本人だけが管理する。

```bash
install -d -m 700 ~/.ssh
ssh-keygen -t ed25519 -a 100 \
  -f ~/.ssh/id_ed25519_i2lab \
  -C "USERNAME@i2lab.test"

cat ~/.ssh/id_ed25519_i2lab.pub
ssh-keygen -lf ~/.ssh/id_ed25519_i2lab.pub -E sha256
```

公開鍵の1行全体と表示された`SHA256:...` fingerprintを管理者へ送る。秘密鍵`~/.ssh/id_ed25519_i2lab`は送らない。

## 3. AWS IAM Identity Centerを設定する

```bash
aws configure sso --profile i2lab
aws sso login --profile i2lab
aws sts get-caller-identity --profile i2lab
```

画面の指示に従って本人のIdentity Center userでsign inし、MFAを登録・認証する。最後のコマンドがAWS accountと自分のassumed-role ARNを表示すれば成功である。

## 4. SSH configを設定する

[研究室内向けSSH設定例](../config/ssh_config.example)の冒頭コメントに従い、`-ssm`のブロックを本人のPCへ取り込む。保存先・置換する値・鍵パス・Windows用の変更点は設定例に記載している。許可されたGPUとSSMの入口を設定する。

## 5. 接続を確認する

```bash
ssh -G quadra-ssm >/dev/null
ssh quadra-ssm 'hostname -f; id; printf "%s\\n" "$SHELL"'
ssh rtx-a6000x2-ssm 'hostname -f; id'
ssh rtx4090-ssm 'hostname -f; id'
ssh rtx5090-ssm 'hostname -f; id'
```

利用許可されたGPUだけを接続する。日常利用、NFS、Docker、VS Code、障害対応は[利用者共通ガイド](remote-access-user-guide.md)を参照する。
