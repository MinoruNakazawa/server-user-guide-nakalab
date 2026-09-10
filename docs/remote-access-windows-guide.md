# Windows 11向けAWS SSM接続の初期設定

本書は研究室内の`-ssm`利用者向けである。`-lan`・`-campus`のみを使う人は[利用マニュアルの入口](README.md)から該当手順へ進む。

この手順は[SSM利用者ガイド](remote-access-user-guide.md)のWindows 11向け初期設定である。管理者からFreeIPA username、AWS IAM Identity CenterのStart URL・SSO region・AWS account・permission set・Quadra Managed Node IDを受け取ってから進める。

## 1. 必要softwareを導入する

PowerShellを**管理者として実行**して、AWS CLI v2、Session Manager Plugin、OpenSSH Clientを導入する。

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi

$installer = "$env:TEMP\SessionManagerPluginSetup.exe"
Invoke-WebRequest `
  https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe `
  -OutFile $installer
Start-Process -FilePath $installer -Verb RunAs -Wait

Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

PowerShellを開き直して確認する。

```powershell
aws --version
session-manager-plugin --version
ssh -V
```

VS Codeを使う場合は、通常のPowerShellで導入する。

```powershell
winget install Microsoft.VisualStudioCode
code --install-extension ms-vscode-remote.remote-ssh
```

## 2. SSH鍵を作成して提出する

[SSH鍵の作成とFreeIPAアカウントへの登録](ssh-key-setup.md)の、このOSに対応する手順を進める。希望名と鍵のコメントの違い、既存鍵の確認、パスフレーズの入力、公開鍵・指紋の提出までを説明している。すでに本人の鍵が登録済みなら再作成しない。

鍵は普段使用する本人のPCアカウントで作成する。管理者へ提出するのは公開鍵と指紋であり、秘密鍵はPCに保管する。SSH設定には希望名ではなく、管理者が確定・通知したFreeIPAユーザー名を使用する。

## 3. AWS IAM Identity Centerを設定する

```powershell
aws configure sso --profile i2lab
aws sso login --profile i2lab
aws sts get-caller-identity --profile i2lab
```

ブラウザで本人のIdentity Center userとしてsign inし、MFAを登録・認証する。

## 4. SSH configを設定する

[研究室内向けSSH設定例](../config/ssh_config.example)の冒頭コメントに従い、`-ssm`のブロックを本人のPCへ取り込む。保存先・置換する値・鍵パス・Windows用の変更点は設定例に記載している。許可されたGPUとSSMの入口を設定する。

## 5. 接続を確認する

通常のPowerShellで実行する。

```powershell
ssh -G quadra-ssm | Out-Null
ssh quadra-ssm 'hostname -f; id; printf "%s\n" "$SHELL"'
ssh rtx-a6000x2-ssm 'hostname -f; id'
ssh rtx4090-ssm 'hostname -f; id'
ssh rtx5090-ssm 'hostname -f; id'
```

利用許可されたGPUだけを接続する。日常利用、NFS、Docker、VS Code、障害対応は[利用者共通ガイド](remote-access-user-guide.md)を参照する。
