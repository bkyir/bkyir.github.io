# GitLab API 常用接口

整理 GitLab API 的常用操作：**上传安装包**、**查询 Package**、**创建 Release**。

所有示例均用占位符表示，使用时替换成你自己的值：

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `<GITLAB_HOST>` | GitLab 服务器地址（含协议+端口） | `https://gitlab.example.com` |
| `<PROJECT_PATH>` | 项目完整路径，`/` 需编码成 `%2F` | `mygroup%2Fmyproject` |
| `<YOUR_PRIVATE_TOKEN>` | GitLab 私人访问令牌 | `glpat-xxxxxxxxxxxxxxxx` |
| `<PACKAGE_NAME>` | 包名（Generic Package 名称） | `my-app` |
| `<VERSION>` | 版本号 | `1.0.0` |
| `<FILE_NAME>` | 上传的文件名 | `my-app-1.0.0.exe` |

> ⚠️ **安全提醒**：**永远不要把真实 token、内网地址等敏感信息提交到公开仓库**。

---

## 1. 上传安装包到 Package Registry

把编译好的安装包传到 Generic Package 仓库：

```bash
curl --header "PRIVATE-TOKEN: <YOUR_PRIVATE_TOKEN>" \
     --upload-file <FILE_NAME> \
     "<GITLAB_HOST>/api/v4/projects/<PROJECT_PATH>/packages/generic/<PACKAGE_NAME>/<VERSION>/<FILE_NAME>"
```

> URL 里的项目路径要把 `/` 编码成 `%2F`。
> Generic Package 的路径格式：`/packages/generic/<包名>/<版本>/<文件名>`。

---

## 2. 查询项目的所有 Package

```bash
curl --header "PRIVATE-TOKEN: <YOUR_PRIVATE_TOKEN>" \
     "<GITLAB_HOST>/api/v4/projects/<PROJECT_PATH>/packages"
```

---

## 3. 创建 Release（发布版本）

发布版本需要先打对应名字的 tag（与 `<VERSION>` 一致），否则会失败：

```bash
curl --request POST \
  --header "PRIVATE-TOKEN: <YOUR_PRIVATE_TOKEN>" \
  --header "Content-Type: application/json" \
  --data '{
    "name": "<PACKAGE_NAME> <VERSION>",
    "tag_name": "<VERSION>",
    "description": "Release <VERSION>"
  }' \
  "<GITLAB_HOST>/api/v4/projects/<PROJECT_PATH>/releases"
```

---

## 📌 参考

- GitLab API 更多接口参考官方文档：https://docs.gitlab.com/ee/api/
- Generic Package Registry：https://docs.gitlab.com/ee/user/packages/generic_packages/
