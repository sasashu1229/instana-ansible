# Ansible Playbook - Sudoパスワード対話式入力ガイド

## 概要
このガイドでは、Instana Agent のインストール・アップグレード用Playbookを、sudoパスワードを対話式で入力して実行する方法を説明します。

## 修正内容
両方のPlaybookファイルに `become_method: sudo` を追加しました：
- `ansible-installv20260202.yaml`
- `ansible-upgrade-20260202.yaml`

## 実行方法

### 1. インストール用Playbook（対話式パスワード入力）
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml --ask-become-pass
```

または短縮形：
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml -K
```

### 2. アップグレード用Playbook（対話式パスワード入力）
```bash
ansible-playbook -i inventory.ini ansible-upgrade-20260202.yaml --ask-become-pass
```

または短縮形：
```bash
ansible-playbook -i inventory.ini ansible-upgrade-20260202.yaml -K
```

## オプション説明

### `--ask-become-pass` または `-K`
- sudoパスワードを対話式で入力するためのオプション
- Playbook実行前にパスワード入力プロンプトが表示されます
- 入力したパスワードは、Playbook内の全ての `become: true` タスクで使用されます

### その他の便利なオプション

#### `-i inventory.ini`
- インベントリファイルを指定（ホスト情報）

#### `--check`
- ドライラン（実際には変更を行わない）
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml -K --check
```

#### `-v`, `-vv`, `-vvv`
- 詳細度を上げる（デバッグ用）
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml -K -vv
```

#### `--limit`
- 特定のホストのみに実行を制限
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml -K --limit server1
```

## パスワードなしsudo設定（オプション）

対話式入力を避けたい場合は、管理対象サーバーでパスワードなしsudoを設定できます：

```bash
# 管理対象サーバーで実行
sudo visudo
```

以下の行を追加（ユーザー名を適宜変更）：
```
your_username ALL=(ALL) NOPASSWD: ALL
```

この設定を行った場合、`-K` オプションなしでPlaybookを実行できます：
```bash
ansible-playbook -i inventory.ini ansible-installv20260202.yaml
```

## トラブルシューティング

### パスワードが正しいのに認証エラーが出る場合
- SSH接続が正しく設定されているか確認
- インベントリファイルのホスト設定を確認
- `-vvv` オプションで詳細ログを確認

### "Missing sudo password" エラー
- `-K` または `--ask-become-pass` オプションを付けて実行してください

### タイムアウトエラー
- ネットワーク接続を確認
- `ansible.cfg` でタイムアウト値を調整

## 参考情報
- Ansible公式ドキュメント: https://docs.ansible.com/
- Privilege Escalation: https://docs.ansible.com/ansible/latest/user_guide/become.html