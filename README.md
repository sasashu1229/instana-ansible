# instana-agent-ansible-role

Instana Agent を RPM で導入し、設定ファイル（3点）を配布して稼働確認まで行う Ansible Role です。

## Features
- インストール済みならスキップ
- RPMをリモートへコピーして `rpm -Uvh` で導入
- 設定ファイルのバックアップ採取
- 3つの設定ファイルを配布
- 差分確認（diff）
- Javaプロセス稼働確認（pgrep）

## Prerequisites
- 対象ホストにSSH到達できること
- sudo権限があること
- RPMと設定ファイルを手元に用意できること（Public repoには含めません）

## Repository Structure
- `playbooks/install_instana_agent.yml`: 実行用Playbook
- `roles/instana_agent`: Role本体
- `inventory.ini.example`: インベントリ例（実ファイルはコミットしない）

## How to use

### 1) inventory.ini を作成
```bash
cp inventory.ini.example inventory.ini
# inventory.ini を自環境に合わせて編集
``
