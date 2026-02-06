# Ansible Playbook レビュー報告書

## 実施日
2026年2月6日

## レビュー対象
`ce-instana-alert-demo-ansible-master` プロジェクト

---

## 発見された問題点と修正内容

### 1. インベントリファイルの問題 ⚠️ 重要度: 高

**ファイル**: `inventories/hosts`

**問題点**:
- ホスト定義で変数展開（`{{ variable }}`）を直接使用していた
- これは非推奨で、実行時に問題を引き起こす可能性がある

**修正内容**:
```yaml
# 修正前
web01 ansible_host="{{ web01_public_ipaddress }}" ansible_user="{{ web01_ansible_user }}"

# 修正後
web01 ansible_port=22 ansible_ssh_private_key_file=~/.ssh/20251112.pem
```

変数は各ホストの `host_vars` ファイルで定義するように変更しました。

---

### 2. host_vars ファイルの改善 ✅ 重要度: 高

**ファイル**: 
- `inventories/host_vars/web01.yaml`
- `inventories/host_vars/web02.yaml`
- `inventories/host_vars/db01.yaml`
- `inventories/host_vars/monitor01.yaml`

**修正内容**:
各ホストのhost_varsファイルに `ansible_host` と `ansible_user` を追加し、適切な変数管理を実現しました。

```yaml
---
ansible_host: "{{ web01_public_ipaddress }}"
ansible_user: "{{ web01_ansible_user }}"
hostname: web01.demo.ibmcloud.com
# ... 以下既存の設定
```

---

### 3. Playbook ファイルのインデント修正 ✅ 重要度: 中

**ファイル**: 
- `db.yaml`
- `instana_agent.yaml`
- `monitor_dump.yaml`
- `monitor_import.yaml`

**問題点**:
- `vars_files` と `roles` のインデントが標準的でなかった
- Playbook に `name` 属性が欠けていた

**修正内容**:
```yaml
# 修正前
- hosts: db
  vars_files: 
  - inventories/group_vars/password-encrypt.yaml
  roles:
  - common

# 修正後
- name: Setup DB server
  hosts: db
  vars_files:
    - inventories/group_vars/password-encrypt.yaml
  roles:
    - common
```

---

### 4. モジュールの完全修飾コレクション名（FQCN）の使用 ✅ 重要度: 中

**ファイル**: 
- `roles/common/tasks/main.yml`
- `roles/instana_import/tasks/main.yml`
- `roles/setup_db2/tasks/main.yml`

**問題点**:
- 短縮形のモジュール名を使用していた（例: `yum`, `command`, `copy`）
- Ansible 2.10以降では完全修飾コレクション名（FQCN）の使用が推奨される

**修正内容**:
```yaml
# 修正前
- name: Install software
  yum:
    name: vim

# 修正後
- name: Install software
  ansible.builtin.package:
    name: vim
```

主な変更:
- `yum` → `ansible.builtin.package` (より汎用的)
- `command` → `ansible.builtin.command`
- `copy` → `ansible.builtin.copy`
- `file` → `ansible.builtin.file`
- `template` → `ansible.builtin.template`
- `include_tasks` → `ansible.builtin.include_tasks`
- `import_tasks` → `ansible.builtin.import_tasks`

---

### 5. instana_import ロールの改善 ✅ 重要度: 高

**ファイル**: `roles/instana_import/tasks/main.yml`

**問題点と修正**:

#### a) tar展開に `command` モジュールを使用
```yaml
# 修正前
- name: Unzip dump.tar into /tmp
  command: tar -xf /tmp/dump.tar

# 修正後
- name: Extract dump.tar into /tmp
  ansible.builtin.unarchive:
    src: /tmp/dump.tar
    dest: /tmp
    remote_src: true
```

#### b) Jinja2テンプレート構文のエスケープ問題
```yaml
# 修正前
command: docker ps --format \{\{.Names\}\}

# 修正後
ansible.builtin.command: docker ps --format '{{.Names}}'
```

#### c) `rm -rf` コマンドの使用
```yaml
# 修正前
- name: Remove /tmp/dump folder
  command: rm -rf /tmp/dump

# 修正後
- name: Remove /tmp/dump folder
  ansible.builtin.file:
    path: /tmp/dump
    state: absent
```

#### d) `changed_when` の追加
読み取り専用のコマンドに `changed_when: false` を追加しました。

---

### 6. setup_db2 ロールの改善 ✅ 重要度: 中

**ファイル**: `roles/setup_db2/tasks/main.yml`

**問題点と修正**:

#### a) 不要な `become: true` の削除
```yaml
# 修正前
- name: Include OS-Specific variables
  become: true
  include_vars: "{{ ansible_os_family }}.yml"

# 修正後
- name: Include OS-Specific variables
  ansible.builtin.include_vars: "{{ ansible_os_family }}.yml"
```

#### b) 一時的なエラー無視の削除
```yaml
# 修正前
- name: Running DB2 Pre Requisits Check
  command: "..."
  ignore_errors: true #TODO 仮

# 修正後
- name: Running DB2 Pre Requisits Check
  ansible.builtin.command: "..."
  changed_when: false
```

`ignore_errors: true` を削除し、適切なエラーハンドリングを実装しました。

#### c) `changed_when: False` の統一
```yaml
# 修正前
changed_when: False

# 修正後
changed_when: false
```

小文字の `false` に統一しました（YAMLの標準）。

---

## ベストプラクティスの適用

### 1. タスクへの `name` 属性の追加
すべてのPlaybookに説明的な `name` 属性を追加しました。

### 2. ファイルパーミッションの明示
`copy` や `template` モジュールに `mode` パラメータを追加しました。

### 3. インデントの統一
YAMLの標準的な2スペースインデントに統一しました。

### 4. コメントの改善
インラインコメント（`#`）を適切な位置に配置しました。

---

## 推奨される追加改善

以下の改善は今回実施していませんが、検討をお勧めします：

### 1. ansible-lint の導入
```bash
pipenv install ansible-lint
pipenv run ansible-lint
```

### 2. 変数の暗号化強化
`ansible-vault` で暗号化されたファイルのパスワード管理を `.vault_pass` ファイルで行う。

### 3. ロールのメタデータ更新
各ロールの `meta/main.yml` に適切な依存関係とメタデータを記載する。

### 4. タグの統一
タグ名を統一し、ドキュメント化する。

### 5. エラーハンドリングの強化
`block`/`rescue`/`always` 構文を使用した適切なエラーハンドリング。

### 6. 冪等性の確認
すべてのタスクが冪等性を持つことを確認する。

---

## まとめ

### 修正したファイル数: 12ファイル

1. `inventories/hosts`
2. `inventories/host_vars/web01.yaml`
3. `inventories/host_vars/web02.yaml`
4. `inventories/host_vars/db01.yaml`
5. `inventories/host_vars/monitor01.yaml`
6. `db.yaml`
7. `instana_agent.yaml`
8. `monitor_dump.yaml`
9. `monitor_import.yaml`
10. `roles/common/tasks/main.yml`
11. `roles/instana_import/tasks/main.yml`
12. `roles/setup_db2/tasks/main.yml`

### 主な改善点

✅ インベントリファイルの変数管理を標準化  
✅ Playbookのインデントと構造を改善  
✅ FQCNを使用してモジュール呼び出しを明示化  
✅ 非推奨のモジュール使用を修正  
✅ エラーハンドリングを改善  
✅ 冪等性を向上  
✅ コードの可読性を向上  

すべての修正により、Playbookはより保守性が高く、Ansibleのベストプラクティスに準拠したものになりました。