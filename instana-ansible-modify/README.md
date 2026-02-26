# Instana Agent Ansible Playbook

このディレクトリには、Instana AgentをLinuxサーバーにインストール・アップグレードするためのAnsible playbookが含まれています。

## 📁 ファイル構成

```
.
├── README.md                           # このファイル
├── SUDO_PASSWORD_GUIDE.md              # sudoパスワード設定ガイド
├── ansible-installv20260224.yaml       # 新規インストール用playbook
├── ansible-upgrade-20260224.yaml       # アップグレード用playbook
├── ansible_vars.yaml                   # 変数定義ファイル
├── inventory.ini                       # インベントリファイル（ホスト定義）
└── files/                              # 配布ファイル
    ├── instana-agent-static.x86_64.rpm
    ├── configuration.yaml
    ├── com.instana.agent.main.sender.Backend.cfg
    └── mvn-settings.xml
```

## 🚀 使い方

### 前提条件

- Ansible 2.9以上がインストールされていること
- 対象サーバーへのSSH接続が可能であること
- sudo権限があること

### 1. 新規インストール

**sudoパスワードを入力して実行する場合（推奨）：**

```bash
ansible-playbook -i inventory.ini -K ansible-installv20260224.yaml
```

または

```bash
ansible-playbook -i inventory.ini --ask-become-pass ansible-installv20260224.yaml
```

**パスワードなしでsudoが実行できる環境の場合：**

```bash
ansible-playbook -i inventory.ini ansible-installv20260224.yaml
```

### 2. アップグレード

**sudoパスワードを入力して実行する場合（推奨）：**

```bash
ansible-playbook -i inventory.ini -K ansible-upgrade-20260224.yaml
```

または

```bash
ansible-playbook -i inventory.ini --ask-become-pass ansible-upgrade-20260224.yaml
```

**パスワードなしでsudoが実行できる環境の場合：**

```bash
ansible-playbook -i inventory.ini ansible-upgrade-20260224.yaml
```

### 3. 特定のホストのみ実行

```bash
ansible-playbook -i inventory.ini -K ansible-upgrade-20260224.yaml --limit "ホスト名"
```

### 4. ドライラン（変更を実際に適用せずに確認）

```bash
ansible-playbook -i inventory.ini -K ansible-upgrade-20260224.yaml --check
```

### 💡 sudoパスワードについて

- `-K` または `--ask-become-pass` オプションを付けると、実行時に「BECOME password:」というプロンプトが表示され、sudoパスワードの入力が求められます
- Playbookファイル内には `become: true` と `become_method: sudo` が設定されているため、sudo権限が必要な操作が実行されます
- パスワード入力を求めるかどうかはコマンドラインオプションで制御します
- `-K` オプションを付けない場合は、パスワードなしでsudoが実行できる環境（sudoersでNOPASSWD設定など）が必要です

## ⚙️ 設定

### ansible_vars.yaml

主要な変数を定義します：

```yaml
# 必須
new_rpm_filename: instana-agent-static-j9.x86_64.rpm

# オプション（デフォルト値あり）
rpm_remote_dir: /tmp
backup_dir: /var/backups/instana-agent
instana_agent_java_path: /opt/instana/agent/jvm/bin/java
```

### inventory.ini

管理対象サーバーを定義します：

```ini
[managed-server]
hostname1 or IP
hostname2 or IP

[managed-server:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/key.pem
```

## 🔒 セキュリティのベストプラクティス

### 1. SSH秘密鍵の管理

**推奨方法：**

```bash
# 環境変数を使用
export ANSIBLE_PRIVATE_KEY=/path/to/key.pem
ansible-playbook -i inventory.ini playbook.yaml

# または実行時に指定
ansible-playbook -i inventory.ini playbook.yaml --private-key=/path/to/key.pem
```

### 2. 機密情報の管理

`files/com.instana.agent.main.sender.Backend.cfg` にはAPIキーが含まれています。

**推奨：ansible-vaultで暗号化**

```bash
# ファイルを暗号化
ansible-vault encrypt files/com.instana.agent.main.sender.Backend.cfg

# 実行時に復号化
ansible-playbook -i inventory.ini playbook.yaml --ask-vault-pass
```

## 📝 playbook の動作

### ansible-installv20260224.yaml（新規インストール）

1. ✅ 既にインストール済みかチェック（済みならスキップ）
2. 📦 RPMファイルをリモートサーバーにコピー
3. 🔧 RPMをインストール
4. 💾 デフォルト設定ファイルをバックアップ
5. 📄 カスタム設定ファイルを配置
6. 🔍 設定ファイルの差分を表示
7. ✔️ Javaプロセスが起動しているか確認

### ansible-upgrade-20260224.yaml（アップグレード）

1. ✅ Instana Agentがインストール済みかチェック
2. 🔢 現在のバージョンと新バージョンを比較
3. 💾 既存の設定ファイルをバックアップ（タイムスタンプ付き）
4. 🗑️ 既存のRPMをアンインストール
5. 📦 新しいRPMをインストール
6. 🔍 設定ファイルの差分を表示
7. ✔️ Javaプロセスが起動しているか確認

## 🔧 トラブルシューティング

### エラー: "instana-agent は既にインストールされています"

- `ansible-installv20260224.yaml` は既にインストール済みの場合はスキップします
- アップグレードする場合は `ansible-upgrade-20260224.yaml` を使用してください

### エラー: "Permission denied" または sudo関連のエラー

- `-K` または `--ask-become-pass` オプションを付けて実行してください
- sudoパスワードが正しいか確認してください
- 対象サーバーでsudo権限があるか確認してください

### エラー: "サービスは稼働していません"

1. ログを確認：
   ```bash
   tail -f /opt/instana/agent/data/log/agent.log
   ```

2. 手動でプロセスを確認：
   ```bash
   pgrep -f /opt/instana/agent/jvm/bin/java
   ps aux | grep instana
   ```

3. 設定ファイルを確認：
   ```bash
   cat /opt/instana/agent/etc/instana/configuration.yaml
   cat /opt/instana/agent/etc/instana/com.instana.agent.main.sender.Backend.cfg
   ```

### バックアップからの復元

```bash
# バックアップファイルの確認
ls -la /var/backups/instana-agent/

# 復元例
sudo cp /var/backups/instana-agent/configuration.yaml.20260206T014500 \
        /opt/instana/agent/etc/instana/configuration.yaml
```

## 📊 改善履歴

### 2026-02-26
- ✅ sudoパスワード入力対応の実装
- ✅ README.mdにsudoパスワード実行方法を追加
- ✅ トラブルシューティングセクションの拡充

### 2026-02-06
- ✅ Javaプロセスパスの修正（`/usr2/instana/...` → `/opt/instana/...`）
- ✅ バックアップファイル名にタイムスタンプを追加（同日複数回実行対応）
- ✅ 変数ファイルの構造化とコメント追加
- ✅ インベントリファイルのベストプラクティス適用
- ✅ ドキュメント作成

## 📚 参考資料

- [Ansible Documentation](https://docs.ansible.com/)
- [Instana Agent Documentation](https://www.ibm.com/docs/en/instana-observability)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
