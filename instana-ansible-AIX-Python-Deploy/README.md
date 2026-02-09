# AIX OS7.2 Python 3.9インストール Ansible Playbook

このAnsible Playbookは、AIX OS 7.2環境にPython 3.9をRPMパッケージを使用してインストールするためのものです。

## 概要

- **対象OS**: AIX 7.2
- **Pythonバージョン**: Python 3.9.6
- **パッケージ形式**: RPM
- **ネットワーク**: インターネット接続不要（オフライン環境対応）
- **エラーハンドリング**: 強化版（詳細なエラーメッセージとリトライ機能）

## 前提条件

### Ansibleコントローラー側
- Ansible 2.9以上がインストールされていること
- AIXサーバーへSSH接続が可能であること

### AIXサーバー側
- AIX 7.2がインストールされていること
- RPMパッケージマネージャーがインストールされていること
- root権限またはsudo権限があること
- **Python 2.7以上または3.5以上がインストールされていること**（Ansible実行に必要）
  - 一般的なパス: `/usr/bin/python`, `/opt/freeware/bin/python3`
  - Playbookは自動的に利用可能なPythonを検出します

## ディレクトリ構造

```
ansible-python-aix/
├── README.md                    # このファイル
├── ansible.cfg                  # Ansible設定ファイル（SSH接続設定含む）
├── install_python_aix.yaml      # メインのPlaybook
├── inventory.ini                # インベントリファイル
├── vars/
│   └── python_vars.yaml         # 変数定義ファイル
└── files/
    ├── README.md                # RPMファイル配置の説明
    └── python3.9-3.9.6-1.aix7.1.ppc.rpm  # ここにRPMファイルを配置
```

## セットアップ手順

### 1. RPMパッケージの準備

`files/` ディレクトリに Python RPMパッケージを配置します。

```bash
cp /path/to/python3.9-3.9.6-1.aix7.1.ppc.rpm ./files/
```

### 2. インベントリファイルの編集

`inventory.ini` を編集して、対象のAIXサーバー情報を設定します。

```ini
[aix_servers]
aix-server01 ansible_host=192.168.1.100 ansible_user=root

[aix_servers:vars]
ansible_connection=ssh
ansible_python_interpreter=/usr/bin/python3
```

### 3. 変数ファイルの確認（必要に応じて編集）

`vars/python_vars.yaml` で設定を確認・変更できます。

```yaml
python_rpm_file: "python3.9-3.9.6-1.aix7.1.ppc.rpm"
local_rpm_path: "files"
temp_dir: "/tmp/python_install"
cleanup_temp_files: true
```

## 実行方法

### 基本的な実行

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml
```

### SSH鍵を指定して実行

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml --private-key=/path/to/private_key
```

### パスワード認証で実行

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml --ask-pass
```

### sudo/suパスワードを指定して実行

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml --ask-become-pass
```

### ドライラン（実際には実行しない）

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml --check
```

### 詳細ログ出力

```bash
ansible-playbook -i inventory.ini install_python_aix.yaml -vvv
```

## Playbookの動作

このPlaybookは以下の処理を実行します：

### 0. Python検出とFacts収集
- **利用可能なPythonの自動検出**: AIXサーバー上の既存Pythonを検索
  - 検索パス: `/usr/bin/python`, `/usr/bin/python3`, `/opt/freeware/bin/python`, `/opt/freeware/bin/python3`
- **Pythonインタープリタの設定**: 検出されたPythonをAnsible実行用に設定
- **Factsの収集**: システム情報を収集（Pythonが利用可能な場合）

### 1. 事前チェック
- **AIXバージョンの確認**: `oslevel -s` でAIXのバージョンを確認（リトライ機能付き）
- **AIX 7.2以上の検証**: 対応バージョンであることを確認
- **RPMパッケージマネージャーの確認**: RPMが利用可能かチェック（リトライ機能付き）

### 2. ディスク容量チェック
- **/tmpディレクトリの空き容量確認**: 最低500MB以上の空き容量を確認
- **容量不足時の警告**: 具体的な対処方法を表示

### 3. 作業ディレクトリの準備
- **作業用ディレクトリの作成**: `/tmp/python_install` を作成
- **作成失敗時のエラー詳細表示**: 権限やパスの問題を特定

### 4. RPMファイルの転送
- **ローカルファイルの存在確認**: 転送前にファイルの存在を検証
- **RPMファイルの転送**: ローカルのRPMファイルをAIXサーバーへコピー（3回リトライ）
- **転送後の整合性確認**: ファイルサイズとチェックサムを検証

### 5. 既存インストール状況の確認
- **Python 3.9の検出**: 既にインストールされているか確認
- **既存バージョンの表示**: インストール済みの場合、現在のバージョンを表示

### 6. RPMパッケージのインストール/アップグレード
- **新規インストール**: 未インストールの場合 `rpm -ivh` でインストール（2回リトライ）
- **アップグレード**: インストール済みの場合 `rpm -Uvh` でアップグレード（2回リトライ）
- **詳細なエラー情報**: 失敗時に原因と対処方法を表示

### 7. インストール後の検証
- **Pythonバージョンの確認**: `python3.9 --version` でバージョン確認（3回リトライ）
- **実行パスの確認**: Pythonの実行パスを表示
- **基本動作テスト**: `import sys` で基本的な動作を確認

### 8. クリーンアップ
- **一時ファイルの削除**: オプションで一時ファイルを削除
- **クリーンアップ失敗時の警告**: 削除失敗時に手動削除の案内を表示

## エラーハンドリング機能

このPlaybookには以下のエラーハンドリング機能が実装されています：

### 1. リトライ機能
以下のタスクは自動的にリトライされます：
- AIXバージョン確認（3回、2秒間隔）
- RPMパッケージマネージャー確認（3回、2秒間隔）
- RPMファイル転送（3回、5秒間隔）
- RPMインストール/アップグレード（2回、3秒間隔）
- Pythonバージョン確認（3回、2秒間隔）

### 2. 詳細なエラーメッセージ
各エラーには以下の情報が含まれます：
- エラーの原因
- 終了コード
- 標準出力とエラー出力
- 考えられる原因のリスト
- 具体的な対処方法

### 3. 事前検証
インストール前に以下を検証します：
- AIXバージョンの互換性
- RPMパッケージマネージャーの存在
- ディスク容量の十分性
- ローカルRPMファイルの存在
- 転送後のファイル整合性

### 4. インストール後の検証
インストール後に以下を確認します：
- Pythonバージョンの確認
- 実行パスの確認
- 基本動作テスト（import sys）

## トラブルシューティング

### SSH接続エラー: "Host key verification failed"

このエラーは、SSH接続時にホストキーの検証に失敗した場合に発生します。

**解決方法1: ansible.cfgで自動的に解決（推奨・既に設定済み）**

プロジェクトには既に `ansible.cfg` が含まれており、以下の設定でホストキー検証を無効化しています：

```ini
[defaults]
host_key_checking = False

[ssh_connection]
ssh_args = -o ConnectTimeout=30 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

**解決方法2: 手動でホストキーを追加**

```bash
# ホストキーをknown_hostsに追加
ssh-keyscan -H <AIXサーバーのIPアドレス> >> ~/.ssh/known_hosts

# 例
ssh-keyscan -H 129.40.95.161 >> ~/.ssh/known_hosts
```

**接続テスト**

```bash
# Ansible pingモジュールで接続確認
ansible aix_servers -m ping

# 成功時の出力例
# p1317-pvm1.p1317.cecc.ihost.com | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### ファイル転送エラー: "dd: Requested a write of 32768 bytes, but wrote only..."

このエラーは、リモートホストのホームディレクトリの容量不足が原因です。

**解決方法（既に設定済み）**

`ansible.cfg` で一時ディレクトリを `/tmp` に変更しています：

```ini
[defaults]
remote_tmp = /tmp/.ansible-${USER}/tmp

[ssh_connection]
pipelining = False
transfer_method = scp
```

### RPMパッケージマネージャーがない場合

AIXにRPMがインストールされていない場合は、以下のコマンドでインストールしてください：

```bash
# AIXツールボックスからRPMをインストール
installp -aXYd /path/to/media rpm.rte
```

### SSH接続エラー

```bash
# SSH接続テスト
ssh -i /path/to/key user@aix-server

# known_hostsのクリア（必要な場合）
ssh-keygen -R aix-server-ip
```

### 権限エラー

- `become: yes` が設定されていることを確認
- AIXでは `sudo` の代わりに `su` を使用する場合があります
- `inventory.ini` で `ansible_become_method=su` を設定

### Python実行パスの問題

インストール後、Pythonが見つからない場合：

```bash
# AIXサーバー上で実行
export PATH=/opt/freeware/bin:$PATH
which python3.9

# または、フルパスで実行
/opt/freeware/bin/python3.9 --version
```

**注意**: このPlaybookでは、Python 3.9は `/opt/freeware/bin/python3.9` にインストールされます。

### よくあるエラーと対処方法

#### 1. ディスク容量不足エラー
```
/tmpディレクトリの空き容量が不足しています。
必要容量: 500MB以上
```

**対処方法**:
```bash
# 不要な一時ファイルを削除
find /tmp -type f -mtime +7 -delete

# または別のディレクトリを使用
# vars/python_vars.yaml で temp_dir を変更
```

#### 2. RPMファイル転送エラー
```
RPMファイルの転送に失敗しました。
```

**対処方法**:
```bash
# SSH接続を確認
ssh -v user@aix-server

# ネットワーク接続を確認
ping aix-server

# 手動でファイルを転送してテスト
scp files/python3.9-*.rpm user@aix-server:/tmp/
```

#### 3. RPMインストールエラー
```
Python 3.9のインストールに失敗しました。
```

**対処方法**:
```bash
# 依存関係を確認
rpm -qpR /tmp/python_install/python3.9-*.rpm

# RPMファイルの整合性を確認
rpm -K /tmp/python_install/python3.9-*.rpm

# 既存のPythonを削除してから再インストール
rpm -e python3.9
```

#### 4. Python動作テストエラー
```
Pythonの基本動作テストに失敗しました。
```

**対処方法**:
```bash
# 依存ライブラリを確認
ldd /opt/freeware/bin/python3.9

# ライブラリパスを確認
echo $LIBPATH

# 必要に応じてライブラリパスを設定
export LIBPATH=/opt/freeware/lib:$LIBPATH
```

## カスタマイズ

### 異なるPythonバージョンを使用する場合

1. `vars/python_vars.yaml` の `python_rpm_file` を変更
2. 対応するRPMファイルを `files/` ディレクトリに配置

### インストール先を変更する場合

RPMパッケージのインストール先はパッケージ自体で定義されています。
カスタムインストール先が必要な場合は、RPMの `--prefix` オプションを使用するか、
ソースからビルドすることを検討してください。

## セキュリティ考慮事項

- RPMファイルは信頼できるソースから入手してください
- インベントリファイルに機密情報（パスワード等）を直接記載しないでください
- Ansible Vaultを使用して機密情報を暗号化することを推奨します

```bash
# 機密情報を暗号化
ansible-vault encrypt vars/python_vars.yaml

# 暗号化されたファイルを使用して実行
ansible-playbook -i inventory.ini install_python_aix.yaml --ask-vault-pass
```

## ライセンス

このPlaybookはMITライセンスの下で提供されています。

## サポート

問題が発生した場合は、以下を確認してください：

1. AIXのバージョンが7.2であること
2. RPMパッケージマネージャーがインストールされていること
3. 必要な権限があること
4. RPMファイルが正しく配置されていること

## 参考情報

- [AIX Toolbox for Linux Applications](https://www.ibm.com/support/pages/aix-toolbox-linux-applications)
- [Ansible Documentation](https://docs.ansible.com/)
- [Python on AIX](https://www.python.org/downloads/)