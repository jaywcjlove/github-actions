<h1>Github Actions</h1>

测试 GitHub 工作流

<!-- TOC -->

- [常用实例](#常用实例)
  - [获取版本信息](#获取版本信息)
  - [修改 package.json](#修改-packagejson)
  - [提交到 gh-pages 分支](#提交到-gh-pages-分支)
  - [克隆带有 Submodule 的仓库](#克隆带有-submodule-的仓库)
  - [提交 NPM 包](#提交-npm-包)
  - [提交 docker 镜像](#提交-docker-镜像)
  - [提交 commit 到 master 分支](#提交-commit-到-master-分支)
  - [Node.js](#nodejs)
  - [同步 Gitee](#同步-gitee)
- [环境变量](#环境变量)
  - [默认环境变量](#默认环境变量)
  - [自定义环境变量](#自定义环境变量)
- [Github 上下文](#github-上下文)

<!-- /TOC -->

## 常用实例

### 获取版本信息

```yml
- name: Test
  run: |
    # Strip git ref prefix from version
    echo "${{ github.ref }}"
    # VERSION=$(echo "${{ github.ref }}" | sed -e 's,.*/\(.*\),\1,')

    # # Strip "v" prefix from tag name
    # [[ "${{ github.ref }}" == "refs/tags/"* ]] && VERSION=$(echo $VERSION | sed -e 's/^v//')
    echo "$VERSION"
```


### 修改 package.json

```yml
- name: Modify Version
  shell: bash
  run: |
    node -e 'var pkg = require("./package.json"); pkg.version= (new Date().getFullYear().toString().substr(2)) + "." + (new Date().getMonth() + 1) + "." + (new Date().getDate()); require("fs").writeFileSync("./package.json", JSON.stringify(pkg, null, 2))'
```

### 提交到 gh-pages 分支

```yml
- name: Deploy Web
  uses: peaceiris/actions-gh-pages@v2.5.0
  env:
    ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
    PUBLISH_BRANCH: gh-pages
    PUBLISH_DIR: ./web
```

### 克隆带有 Submodule 的仓库

```yml
- name: Clone sub repository
  shell: bash
  run: |
    auth_header="$(git config --local --get http.https://github.com/.extraheader)"
    # git submodule sync --recursive
    # git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --remote --force --recursive --checkout ant.design
```

### 提交 NPM 包

```yml
- run: npm publish --access public
  env:
    NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}
```

获取 `NPM_TOKEN`，可以通过 [npm](https://www.npmjs.com/settings/wcjiang/tokens) 账号创建 `token`

```shell
npm token list [--json|--parseable] # 查看
npm token create [--read-only] [--cidr=1.1.1.1/24,2.2.2.2/16] # 创建
npm token revoke <id|token> # 撤销
```

### 提交 docker 镜像

```yml
# https://www.basefactor.com/github-actions-docker
- name: Docker login
  run: docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASSWORD }}

- name: Build ant.design image
  run: |
    cd ./ant\.design
    docker build -t ant.design .
- name: Tags & Push docs
  run: |
    # Strip git ref prefix from version
    VERSION=$(echo "${{ github.ref }}" | sed -e 's,.*/\(.*\),\1,')

    # Strip "v" prefix from tag name
    [[ "${{ github.ref }}" == "refs/tags/"* ]] && VERSION=$(echo $VERSION | sed -e 's/^v//')

    docker tag ant.design ${{ secrets.DOCKER_USER }}/ant.design:$VERSION
    docker tag ant.design ${{ secrets.DOCKER_USER }}/ant.design:latest
    docker push ${{ secrets.DOCKER_USER }}/ant.design:$VERSION
    docker push ${{ secrets.DOCKER_USER }}/ant.design:latest
```

### 提交 commit 到 master 分支

```yml
- name: 生成一个文件，并将它提交到 master 分支
  run: |
    # Strip git ref prefix from version
    VERSION=$(echo "${{ github.ref }}" | sed -e 's,.*/\(.*\),\1,')
    COMMIT=released-${VERSION}
    # Strip "v" prefix from tag name
    [[ "${{ github.ref }}" == "refs/tags/"* ]] && VERSION=$(echo $VERSION | sed -e 's/^v//')
    echo "输出版本号：$VERSION"
    # 将版本输出到当前 VERSION 文件中
    echo "$VERSION" > VERSION
    echo "1. 输出Commit：$commit"
    echo "2. Released $VERSION"
    git fetch
    git config --local user.email "action@github.com"
    git config --local user.name "GitHub Action"
    git add .
    git commit -am $COMMIT
    git branch -av
    git pull origin master
- name: 将上面的提交 push 到 master 分支
  uses: ad-m/github-push-action@master
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Node.js

```yml
- name: Setup Node
  uses: actions/setup-node@v1
  with:
    node-version: '10.x'
```

```yml
strategy:
  matrix:
    node-version: [10.x, 12.x, 14.x]

steps:
- uses: actions/checkout@v2
- name: Use Node.js ${{ matrix.node-version }}
  uses: actions/setup-node@v1
  with:
    node-version: ${{ matrix.node-version }}
- run: npm ci
- run: npm run build --if-present
- run: npm test

```

### 同步 Gitee

```yml
- name: Sync to Gitee
  run: |
    mirror() {
      git clone "https://github.com/$1/$2"
      cd "$2"
      git remote add gitee "https://jaywcjlove:${{ secrets.GITEE_TOKEN }}@gitee.com/uiw/$2.git"
      git remote set-head origin -d
      git push gitee --prune +refs/remotes/origin/*:refs/heads/* +refs/tags/*:refs/tags/*
      cd ..
    }
    mirror uiwjs uiw
```

## 环境变量

> [默认环境变量](https://help.github.com/cn/actions/configuring-and-managing-workflows/using-environment-variables#default-environment-variables)

强烈建议操作使用环境变量访问文件系统，而非使用硬编码的文件路径。 GitHub 设置供操作用于所有运行器环境中的环境变量。

### 默认环境变量

环境变量 | 描述
---- | ----
CI |  始终设置为 true。
HOME |  用于存储用户数据的 GitHub 主目录路径。 例如 /github/home。
GITHUB_WORKFLOW | 工作流程的名称。
GITHUB_RUN_ID | 仓库中每个运行的唯一编号。 如果您重新执行工作流程运行，此编号不变。
GITHUB_RUN_NUMBER | 仓库中特定工作流程每个运行的唯一编号。 此编号从 1（对应于工作流程的第一个运行）开始，然后随着每个新的运行而递增。 如果您重新执行工作流程运行，此编号不变。
GITHUB_ACTION | 操作唯一的标识符 (id)。
GITHUB_ACTIONS |  当 GitHub 操作 运行工作流程时，始终设置为 true。 您可以使用此变量来区分测试是在本地运行还是通过 GitHub 操作 运行。
GITHUB_ACTOR |  发起工作流程的个人或应用程序的名称。 例如 octocat。
GITHUB_REPOSITORY | 所有者和仓库名称。 例如 octocat/Hello-World。
GITHUB_EVENT_NAME | 触发工作流程的 web 挂钩事件的名称。
GITHUB_EVENT_PATH | 具有完整 web 挂钩事件有效负载的文件路径。 例如 /github/workflow/event.json。
GITHUB_WORKSPACE |  GitHub 工作空间目录路径。 如果您的工作流程使用 [actions/checkout](https://github.com/actions/checkout) 操作，工作空间目录将包含存储仓库副本的子目录。 如果不使用 [actions/checkout](https://github.com/actions/checkout) 操作，该目录将为空。 例如 /home |/runner/work/my-repo-name/my-repo-name。
GITHUB_SHA |  触发工作流程的提交 SHA。 例如 ffac537e6cbbf934b08745a378932722df287a53。
GITHUB_REF |  触发工作流程的分支或标记参考。 例如 refs/heads/feature-branch-1。 如果分支或标记都不适用于事件类型，则变量不会存在。
GITHUB_HEAD_REF | 仅为复刻的仓库设置。 头部仓库的分支。
GITHUB_BASE_REF | 仅为复刻的仓库设置。 基础仓库的分支。

### 自定义环境变量

> 注： `GitHub` 会保留 `GITHUB_` 环境变量前缀供 `GitHub` 内部使用。 设置有 `GITHUB_` 前缀的环境变量或密码将导致错误。

在 `https://github.com/<用户名>/<项目名称>/settings/secrets` 中添加 `secrets` `NODE_API_TOKEN`，在工作流中设置环境变量 [`NODE_API_TOKEN`](https://github.com/jaywcjlove/github-actions/blob/799b232fca3d9df0272eaa12610f9ebfca51b288/.github/workflows/ci.yml#L46)

```yml
- name: 测试 nodejs 获取环境变量
  env:
    NODE_API_TOKEN: ${{ secrets.NODE_API_TOKEN }}
  run: npm run env
```

## Github 上下文

> [Github 上下文](https://help.github.com/cn/actions/reference/context-and-expression-syntax-for-github-actions)

属性名称 | 类型 | 描述
---- | ---- | ----
github | object | 工作流程中任何作业或步骤期间可用的顶层上下文。
github.event | object | 完整事件 web 挂钩有效负载。 更多信息请参阅“触发工作流程的事件”。
github.event_path | string | 运行器上完整事件 web 挂钩有效负载的路径。
github.workflow | string | 工作流程的名称。 如果工作流程文件未指定 name，此属性的值将是仓库中工作流程文件的完整路径。
github.job | string | 当前作业的 job_id。
github.run_id | string | 仓库中每个运行的唯一编号。 如果您重新执行工作流程运行，此编号不变。
github.run_number | string | 仓库中特定工作流程每个运行的唯一编号。 此编号从 1（对应于工作流程的第一个运行）开始，然后随着每个新的运行而递增。 如果您重新执行工作流程运行，此编号不变。
github.actor | string | 发起工作流程运行的用户的登录名。
github.repository | string | 所有者和仓库名称。 例如 Codertocat/Hello-World。
github.repository_owner | string | 仓库所有者的名称。 例如 Codertocat。
github.event_name | string | 触发工作流程运行的事件的名称。
github.sha | string | 触发工作流程的提交 SHA。
github.ref | string | 触发工作流程的分支或标记参考。
github.head_ref | string | 工作流程运行中拉取请求的 head_ref 或来源分支。 此属性仅在触发工作流程运行的事件为 pull_request 时才可用。
github.base_ref | string | 工作流程运行中拉取请求的 base_ref 或目标分支。 此属性仅在触发工作流程运行的事件为 pull_request 时才可用。
github.token | string | 代表仓库上安装的 GitHub 应用程序进行身份验证的令牌。 这在功能上等同于 GITHUB_TOKEN 密码。 更多信息请参阅“使用 GITHUB_TOKEN 验证身份”。
github.workspace | string | 使用 checkout 操作时步骤的默认工作目录和仓库的默认位置。
github.action | string | 正在运行的操作的名称。 在当前步骤运行脚本时，GitHub 删除特殊字符或使用名称 run。 如果在同一作业中多次使用相同的操作，则名称将包括带有序列号的后缀。 例如，运行的第一个脚本名称为 run1，则第二个脚本将命名为 run2。 同样，actions/checkout 第二次调用时将变成 actionscheckout2。