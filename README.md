# 研究室GPUサーバー 利用者ガイド

**[利用マニュアルの入口](docs/README.md)から、自分の利用区分を選んでください。**

| 利用区分 | マニュアル | 接続方法 |
| --- | --- | --- |
| 研究室内の利用者 | [研究室内利用者の入口](docs/README.md#lab-users) | -lan・-ssm。条件に応じて-campusも利用可能 |
| 研究室外の学内利用者・ゼミ生 | [学内利用者マニュアル](docs/campus-user-guide.md) | -campus（AWS不要） |

## 接続方法

- **-lan**：対象GPUへ直接到達できる研究室LANなどで使用。AWS認証は不要。
- **-campus**：学内踏み台へ到達できるネットワークで使用。AWS認証は不要。
- **-ssm**：AWS利用許可のある研究室内利用者が、学外・自宅などから使用。AWS SSO認証が必要。

利用には本人のアカウント・SSH公開鍵登録と、対象GPUの利用許可が必要です。研究室外の学内利用者にはAWS権限を発行しません。

## SSH設定例

保存先、置換する値、OS別の注意事項、接続名は設定例のコメントに記載しています。

- [直接接続・AWS経由の設定例](config/ssh_config.example)
- [学内踏み台経由の設定例](config/ssh_config.campus.example)

## 問い合わせ

中沢実：[nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp)

本リポジトリは申請・接続・動作確認など利用者向けの情報を掲載します。利用者情報や秘密鍵、パスワードをIssue・Pull Request・Gitへ投稿しないでください。
