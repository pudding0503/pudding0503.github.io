## 布丁工坊部署

### 1. 生成部署密钥

一路按回车直到生成成功：

```ssh
ssh-keygen -f github-deploy-key
```

当前目录下会有 `github-deploy-key` 和 `github-deploy-key.pub` 两个文件。

### 2. 配置部署密钥

#### 2.1 添加 key

复制 `github-deploy-key` 文件内容，在 `blog` 仓库 `Settings -> Secrets -> Add a new secret` 页面上添加：

1. 在 `Name` 输入框填写 `HEXO_DEPLOY_PRI`。
2. 在 `Value` 输入框填写 `github-deploy-key` 文件内容。

#### 2.2 添加 key pub

复制 `github-deploy-key.pub` 文件内容，在 `pudding.github.io` 仓库 `Settings -> Deploy keys -> Add deploy key` 页面上添加：

1. 在 `Title` 输入框填写 `HEXO_DEPLOY_PUB`。
2. 在 `Key` 输入框填写 `github-deploy-key.pub` 文件内容。
3. 勾选 `Allow write access` 选项。

### 3. 编写 Github Actions

在 `blog` 仓库根目录下创建 `.github/workflows/deploy.yml` 文件，目录结构如下：

```yaml
name: Pudding Blog Build

# https://chee5e.space/hexo-deploy-github-actions/
# https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/

on:
  push:
    branches:
      - main

env:
  GIT_USER: Haoning Wu
  GIT_EMAIL: whn@nousbuild.com
  THEME_REPO: pudding0503/hexo-theme-fluid
  THEME_BRANCH: master
  DEPLOY_REPO: pudding0503/pudding0503.github.io
  DEPLOY_BRANCH: main

jobs:
  build:
    name: Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ubuntu-latest]
        node_version: [14.x]

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Checkout theme repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.THEME_REPO }}
          ref: ${{ env.THEME_BRANCH }}
          path: themes/fluid

      - name: Checkout deploy repo
        uses: actions/checkout@v2
        with:
          repository: ${{ env.DEPLOY_REPO }}
          ref: ${{ env.DEPLOY_BRANCH }}
          path: .deploy_git

      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node_version }}

      - name: Configuration environment
        env:
          HEXO_DEPLOY_PRI: ${{secrets.HEXO_DEPLOY_PRI}}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL
          cp _config.fluid.yml themes/fluid/_config.yml

      - name: Install dependencies
        run: |
          npm install

      - name: Deploy hexo
        run: |
          npm run deploy
```

### 4. 其他工作

#### 4.1 添加 hexo-deployer-git 依赖

在 `package.json` 中添加如下依赖，才可正常部署：

```json
{
  "dependencies": {
    "hexo-deployer-git": "^3.0.0"
  }
}
```

#### 4.2 修改为 git 方式

在 `_config.yml` 中修改为 git 方式：

```yaml
deploy:
  type: git
  repo: git@github.com:pudding0503/pudding0503.github.io.git
  branch: main
```

#### 4.3 主题配置

复制一份 `_config.yml` 到根目录，重命名为 `_config.fluid.yml`。