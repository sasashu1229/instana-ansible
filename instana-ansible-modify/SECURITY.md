# セキュリティガイドライン

## ⚠️ 重要な注意事項

このAnsibleプロジェクトには機密情報が含まれています。適切に管理してください。

## 🔐 機密情報を含むファイル

### 1. com.instana.agent.main.sender.Backend.cfg

**含まれる情報：**
- Instana SaaS APIキー（`key=O_xPXPnaToa9ZpkiPG0Elg`）
- 接続先ホスト情報

**推奨対策：**

#### オプションA: Ansible Vaultで暗号化

```bash
# ファイルを暗号化
ansible-vault encrypt files/com.instana.agent.main.sender.Backend.cfg

# 実行時
ansible-playbook -i inventory.ini playbook.yaml --ask-vault-pass

# またはパスワードファイルを使用
ansible-playbook -i inventory.ini playbook.yaml --vault-password-file ~/.vault_pass
```

#### オプションB: 環境変数を使用

playbookを修正して、APIキーを環境変数から取得：

```yaml
- name: Replace Backend.cfg with template
  ansible.builtin.template:
    src: "files/com.instana.agent.main.sender.Backend.cfg.j2"
    dest: /opt/instana/agent/etc/instana/com.instana.agent.main.sender.Backend.cfg
```

テンプレートファイル（`.j2`）内：
```
key={{ lookup('env', 'INSTANA_API_KEY') }}
```

### 2. inventory.ini

**含まれる情報：**
- SSH秘密鍵のパス
- ホスト名/IPアドレス
- ユーザー名

**推奨対策：**

```bash
# 実行時に秘密鍵を指定
ansible-playbook -i inventory.ini playbook.yaml --private-key=/path/to/key.pem

# または環境変数を使用
export ANSIBLE_PRIVATE_KEY=/path/to/key.pem
```

### 3. SSH秘密鍵ファイル（.pem）

**推奨対策：**
- Gitリポジトリに含めない
- パーミッションを `600` に設定
- パスフレーズで保護

```bash
chmod 600 ~/20251112.pem
```

## 📋 セキュリティチェックリスト

- [ ] `.gitignore` を設定して機密ファイルをコミットしない
- [ ] `com.instana.agent.main.sender.Backend.cfg` を暗号化または環境変数化
- [ ] SSH秘密鍵のパーミッションを確認（600）
- [ ] 本番環境では ansible-vault を使用
- [ ] APIキーを定期的にローテーション
- [ ] 不要なバックアップファイルを削除
- [ ] ログファイルに機密情報が含まれていないか確認

## 🔄 APIキーのローテーション

Instana APIキーを変更した場合：

1. Instana UIで新しいキーを生成
2. `files/com.instana.agent.main.sender.Backend.cfg` を更新
3. playbookを再実行してエージェントに反映

## 🚨 インシデント対応

もし機密情報が漏洩した場合：

1. **即座に対応：**
   - Instana UIでAPIキーを無効化
   - 新しいAPIキーを生成
   - SSH秘密鍵を変更

2. **影響範囲の確認：**
   - Gitの履歴を確認
   - アクセスログを確認

3. **再発防止：**
   - このドキュメントのベストプラクティスを実施
   - チームメンバーに周知

## 📚 参考資料

- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Ansible Security Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#best-practices-for-variables-and-vaults)
- [IBM Instana Security](https://www.ibm.com/docs/en/instana-observability)