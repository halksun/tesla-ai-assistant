# 02 — 公钥生成与 GitHub Pages 托管

Tesla 要求 Partner 公钥发布在：

```text
https://<your-domain>/.well-known/appspecific/com.tesla.3p.public-key.pem
```

## 生成密钥对（在私密目录，勿提交私钥）

可使用 upstream 脚本，或 OpenSSL 生成 P-256 密钥对。私钥仅留在车主本机安全位置；本仓库只允许出现「如何托管公钥」的说明。

```bash
# 示例：使用 upstream 仓库中的脚本（路径按你的 clone 位置调整）
scripts/generate-keypair.sh "$HOME/tesla-partner-keys"
```

## GitHub Pages 布局示例

```text
.nojekyll
.well-known/appspecific/com.tesla.3p.public-key.pem   # 仅公钥
callback.html                                         # 可选 OAuth 粘贴回退页
README.md                                             # 说明这是公钥托管页
```

要点：

- 必须有 `.nojekyll`，否则 `.well-known` 可能被忽略。
- **只上传公钥**。私钥永不进 Pages 仓库。
- 每个应用使用**自己的**密钥对；不要复用他人仓库里的公钥。
- `github.io` 是否总被 Tesla 接受，以实际 `register-partner` 结果为准；失败时改用自有域名。

公钥托管仓库与本 playbook 仓库应分离：前者极简且几乎不改；后者是文档。
