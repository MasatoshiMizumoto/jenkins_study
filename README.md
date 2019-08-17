jenkins_study
====

## What is this?

エンジニアリングスクール[RaiseTech](https://raise-tech.net/)のAWSフルコース第14回以降講座の課題：「Jenkinsを使っての環境構築」を実施したものです。
- 本課題に関して詳細な手順書等のドキュメントはスクールより提供されておらず、AWS公式ドキュメントや検索サイトでヒットする情報を元に作成しています。

## 機能
- CloudFormationテンプレートでのリソース構築（[AWS提供テンプレート](https://s3-ap-northeast-1.amazonaws.com/cloudformation-templates-ap-northeast-1/EIP_With_Association.template)を参考にカスタマイズ）
- Ansibleでのインストール自動化[ansible_study](https://github.com/miima17/ansible_study)＋ec2.pyでのhosts決定

## 構成
- ディレクトリ構成

```
├── README.md
├── ansible
│   ├── ansible.cfg
│   ├── inventory
│   │   ├── ec2.ini
│   │   └── ec2.py
│   ├── playbooks
│   │   └── playbook.yml
│   └── roles
│       ├── common
│       │   ├── defaults
│       │   │   └── main.yml
│       │   └── tasks
│       │       └── main.yml
│       ├── nginx
│       │   ├── defaults
│       │   │   └── main.yml
│       │   └── tasks
│       │       └── main.yml
│       ├── nodejs
│       │   ├── defaults
│       │   │   └── main.yml
│       │   ├── tasks
│       │   │   └── main.yml
│       │   └── templates
│       │       └── npm.sh.j2
│       ├── rails
│       │   ├── README.md
│       │   ├── defaults
│       │   │   └── main.yml
│       │   ├── tasks
│       │   │   └── main.yml
│       │   └── templates
│       │       ├── init.d.unicorn.j2
│       │       ├── nginx.conf.j2
│       │       └── unicorn.rb.j2
│       └── rbenv
│           ├── defaults
│           │   └── main.yml
│           └── tasks
│               └── main.yml
└── cfn_template
    └── ec2_with_eip.yaml
```

## 作成環境

- サーバ1（構築済）Jenkins:2.189
- サーバ2（構築済）ansible:2.8.3
- 自動化対象ホスト:Amzaon Linux 2（今回のジョブで作製）
- 作業リージョン：アジアパシフィック（東京）


## 導入

###事前準備

- 最新のAmazon Linuxイメージで構築したyum updateをかけて、サーバ2でssh-keygenした共通鍵を登録したものをAMIとして作製しておきます。
- サーバ1,2はAWS CLIを正常に実行できるように準備しておいてください。
- サーバ2ではec2.pyの実行環境が必要です。

Jenkinsで1つ目のジョブを作製します。
- リポジトリをgit clone
- aws cliから作製予定のスタックと同じ名称のスタックをdeleteする。（やり直しなどの為。本来はupdateが良い。）
- aws cliでcfn_template内のyamlを使いスタックをcreateする。

Jenkinsで2つ目のジョブを作製します。（全てAnsibleサーバに対してのリモートシェル）
- リポジトリをgit clone
- ec2.pyに実行権付与（chmod +x)
- ec2.pyの動作確認。
```
inventory/ec2.py -refresh-chache
```
- ansible-playbookで対象ホストが判定されるか確認。
```
ansible-playbook -i inventory/ec2.py -l "ami-00a5245b4816c38e6" playbook/playbook.yml --list-hosts
```
- ansible-playbookの実行。（秘密鍵と接続ユーザ名が必要）
```
ansible-playbook -i inventory/ec2.py -l "ami-00a5245b4816c38e6" playbook/playbook.yml --private-key="~/.ssh/ansible" -u ec2-user
```

## 参考サイト

- [ec2.py、EC2.ini(ansibleのGithubより)](https://github.com/ansible/ansible/tree/devel/contrib/inventory)
- [[AWS]AnsibleのDynamic Inventoryを使って実行対象のEC2をタグ等で柔軟に指定する](https://dev.classmethod.jp/cloud/aws/ansible-dynamic-inventory-2/)
- [Ansible statモジュールで取得できる内容 - Qiita](https://qiita.com/tiibun/items/02897ef788d21f3f05e0)
- [AWSにJenkins環境を構築する - Qiita](https://qiita.com/hitomatagi/items/4bf578b46c525fc01514)

## 作成者

[@miima_17](https://twitter.com/miima_17)