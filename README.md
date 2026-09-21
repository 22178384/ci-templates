# ci-templates

> 可直接复制到项目里的 GitHub Actions 工作流模板。

## 模板
- [python-ci.yml](workflows/python-ci.yml)：Python 测试 + 覆盖率
- [node-ci.yml](workflows/node-ci.yml)：Node 安装 / 构建 / 测试

## 用法
把 `workflows/` 下的文件复制到你的仓库的 `.github/workflows/` 目录即可启用，按需改 Python / Node 版本。

## 生态联动
- 种子项目（含 CI 思路）→ [@22178384/project-seed](https://github.com/22178384/project-seed)
- 工具库 → [@c991china/python-utils](https://github.com/c991china/python-utils)
