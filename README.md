# 研究室GPUサーバー 利用者ガイド

**[利用マニュアルの入口](docs/README.md)から、自分の利用区分を選んでください。**

初めて利用する方は、利用区分に応じたMicrosoft Formsから申請してください。

- 研究室内の利用者：[利用申請フォーム](https://forms.cloud.microsoft/r/mgFXy680J1)
- 研究室外の学内利用者・ゼミ生：[利用申請フォーム](https://forms.cloud.microsoft/pages/responsepage.aspx?id=Xbxum3IjmUClZWULy8MnT6rI0kF_K9lEhfaBpChXa5JUNzlWOE05NlQxWVFDNlExMVhRV09BT05YSS4u&route=shorturl)

申請前に[SSH鍵の準備](docs/ssh-key-setup.md)を確認し、公開鍵とFingerPrint（指紋）を用意してください。学外から利用する場合は、申請時にremote-VPNを使う予定も伝えてください。

| 利用区分 | マニュアル | 接続方法 |
| --- | --- | --- |
| 研究室内の利用者 | [研究室内利用者の入口](docs/README.md#lab-users) | -lan。到達性に応じて-campusも利用可能 |
| 研究室外の学内利用者・ゼミ生 | [学内利用者マニュアル](docs/campus-user-guide.md) | -campus |

初めての方は、[システム全体と接続経路](docs/system-overview.md)で仕組みを確認し、[SSH鍵の作成とFreeIPA登録](docs/ssh-key-setup.md)へ進んでください。希望ユーザー名と鍵の関係、OS別の作成方法、公開鍵の提出から接続までを説明しています。

## 接続方法

- **-lan**：対象GPUのSSHへ直接到達できる場合に使用。
- **-campus**：学内踏み台のSSHへ到達できる場合に使用。

**学外・自宅からは、大学の[remote-VPN利用案内](https://uranus.mars.kanazawa-it.ac.jp/dpc/navi_network/navi_network_5/)に従ってVPNへ接続してからSSHを使います。** VPN接続後、対象GPUへ直接到達できれば `-lan`、学内踏み台へ到達できれば `-campus` を選びます。到達できる経路はVPNの設定・大学のネットワーク条件によります。どちらも届かない場合は管理者へ連絡してください。AWS経由の `-ssm` は使用しません。

学内踏み台では中継専用ユーザー `campus-relay` を使い、利用者ごとの公開鍵で認証します。専用のリバースSSHトンネルでQuadraへ中継し、Quadra・GPUではFreeIPAで一元管理された本人の研究室アカウントを使用します。詳しくは[構成と認証の説明](docs/system-overview.md#3--campus学内踏み台とリバーストンネル)を参照してください。

利用には本人のアカウント・SSH公開鍵登録と、対象GPUの利用許可が必要です。

Dockerを利用する方は、接続後に[GPUサーバーのDocker利用ガイド](docs/docker-user-guide.md)で権限・研究環境・データ保存先を確認してください。

## SSH設定例

保存先、置換する値、OS別の注意事項、接続名は設定例のコメントに記載しています。

- [直接接続の設定例](config/ssh_config.example)
- [学内踏み台経由の設定例](config/ssh_config.campus.example)

## 問い合わせ

中沢実：[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)

本リポジトリは申請・接続・動作確認など利用者向けの情報を掲載します。利用者情報や秘密鍵、パスワードをIssue・Pull Request・Gitへ投稿しないでください。
