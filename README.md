<!--idoc:ignore:start-->
Github Actions
===
<!--idoc:ignore:end-->

<!--idoc:config:
title: Github Actions
site: Github Actions
editButton: 
  label: Edit this page on GitHub
  url: https://github.com/jaywcjlove/github-actions/blob/master/
footer: |
  Released under the MIT License. Copyright © 2022 Kenny Wong<br />
  Generated by <a href="https://github.com/jaywcjlove/idoc" target="_blank">idoc</a> v{{idocVersion}}
-->

[![Buy me a coffee](https://img.shields.io/badge/Buy%20me%20a%20coffee-048754?logo=buymeacoffee)](https://jaywcjlove.github.io/#/sponsor)

测试 GitHub 工作流，一些比较特殊的玩儿法。

<!--idoc:ignore:start-->
## 目录

<!-- TOC -->

- [基本概念](#基本概念)
  - [配置文件](#配置文件)
  - [多项任务](#多项任务)
  - [多项任务依赖关系](#多项任务依赖关系)
  - [指定每项任务的虚拟机环境](#指定每项任务的虚拟机环境)
  - [一个工作流完成触发另外一个工作流](#一个工作流完成触发另外一个工作流)
- [常用实例](#常用实例)
  - [获取版本信息](#获取版本信息)
  - [获取是否存在 Tag](#获取是否存在-tag)
  - [修改 package.json](#修改-packagejson)
  - [提交到 gh-pages 分支](#提交到-gh-pages-分支)
  - [克隆带有 Submodule 的仓库](#克隆带有-submodule-的仓库)
  - [步骤依赖作业](#步骤依赖作业)
  - [步骤作业文件共享](#步骤作业文件共享)
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
<!--idoc:ignore:end-->

## 基本概念

GitHub actions 有四个基本的概念，如下：

- workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。
- job （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。
- step（步骤）：每个 job 由多个 step 构成，一步步完成。
- action （动作）：每个 step 可以依次执行一个或多个命令（action）。

### 配置文件

GitHub Actions 采用 [`YAML`](https://www.ruanyifeng.com/blog/2016/07/yaml.html) 格式的配置文件叫做 workflow 文件，存放在代码仓库的 `.github/workflows` 目录。文件名可以任意取，但是后缀名统一为 `.yml`，比如 `ci.yml`。一个库可以有多个 `workflow` 文件。GitHub 只要发现 `.github/workflows` 目录里面有 `.yml` 文件，就会根据配置事件自动运行该文件。

```yml
name: GitHub Actions Demo
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16

      - run: npm install

      - run: npm run build
```

### 定触条件

[`on`](https://docs.github.com/cn/actions/using-workflows/events-that-trigger-workflows) 字段指定触发 `workflow` 的条件，通常是某些事件。

```yml
# push 事件触发 workflow
on: push

# push 事件或 pull_request 事件都可以触发 workflow
on: [push, pull_request]

# 只有在 main 分支 push 事件触发 workflow
on:
  push:
    branches:
      - main

# push 事件触发 workflow，但是 docs 目录下的更改 push 事件不触发 workflow
on:
  push:
    paths-ignore:
      - 'docs/**'
# push 事件触发 workflow，
# 包括 sub-project 目录或其子目录中的文件，触发 workflow
# 除非该文件在 sub-project/docs 目录中，不触发 workflow
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
# 版本发布为 published 时运行工作流程。
on:
  release:
    types: [published]
```

### 多项任务

通过 `jobs` (`jobs.<job_id>.name`)字段，配置一项或多项需要执行的任务。

```yml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

### 多项任务依赖关系

通过 `needs` (`jobs.<job_id>.needs`)字段，指定当前任务的依赖关系。

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

上面配置中，`job1` 必须先于 `job2` 完成，而 `job3` 等待 `job1` 和 `job2` 的完成才能运行。因此，这个 workflow 的运行顺序依次为：`job1`、`job2`、`job3`。

### 多项任务传递参数

```yml
jobs:
  job1:
    runs-on: ubuntu-latest
    # Map a step output to a job output
    outputs:
      output1: ${{ steps.step1.outputs.test }}
      output2: ${{ steps.step2.outputs.test }}
    steps:
      - id: step1
        run: echo "::set-output name=test::hello"
      - id: step2
        run: echo "::set-output name=test::world"
  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - run: echo ${{needs.job1.outputs.output1}} ${{needs.job1.outputs.output2}}
```

### 指定每项任务的虚拟机环境

`runs-on` 字段指定运行所需要的虚拟机环境。⚠️ 它是必填字段。

```yml
runs-on: ubuntu-18.04
```

```yml
jobs:
  build:
    runs-on: ubuntu-18.04
```

| **虚拟环境**             | **YAML 工作流程标签**                   | **注：**                                                                             |
| -------------------- | --------------------------------- | ---------------------------------------------------------------------------------- |
| Windows Server 2022  | `windows-latest` 或 `windows-2022` | `windows-latest` 标签目前使用的是 Windows Server 2022 运行器镜像 |
| Windows Server 2019  | `windows-2019`                    |                                                                                    |
| Ubuntu 22.04         | `ubuntu-22.04`                    | Ubuntu 22.04 目前处于公开测试阶段。                                          |
| Ubuntu 20.04         | `ubuntu-latest` 或 `ubuntu-20.04`  |                                                                                    |
| Ubuntu 18.04         | `ubuntu-18.04`                    |                                                                                    |
| macOS Monterey 12    | `macos-12`                        | macOS 12 目前处于公开测试阶段。                                     |
| macOS Big Sur 11     | `macos-latest` 或 `macos-11`       | `macos-latest` 标签当前使用 macOS 11 运行器镜像。                |
| macOS Catalina 10.15 | `macos-10.15`                     |                                                                                    |

### 指定每项任务的步骤

`steps` 字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

```bash
jobs.<job_id>.steps.name：步骤名称。
jobs.<job_id>.steps.run：该步骤运行的命令或者 action。
jobs.<job_id>.steps.env：该步骤所需的环境变量。
```

```yml
jobs:
  build:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16
          registry-url: 'https://registry.npmjs.org'

      - run: npm install

      - run: npm run build
```

### 一个工作流完成触发另外一个工作流

```yml
# ci.yml
name: Node.js CI
on: push

jobs:
  test:
    # Containers must run in Linux based operating systems
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Install dependencies
        run: npm install

  trigger_tests:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Trigger tests
        uses: actions/github-script@v6
        with:
          script: |
            const res = await github.rest.repos.createDispatchEvent({
              owner: 'jaywcjlove',
              repo: 'typenexus',
              event_type: 'run-deploy'
            });
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

上面 `ci.yml` 工作流执行完成之后，触发 `main.yml`，注意上面的 `run-deploy` 取名保持一致

```yml
# main.yml
name: deploy
on:
  repository_dispatch:
    types: [run-deploy]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
```

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


```yml
name: CI
on:
  push:
    tags:
      - v*

jobs:
  create-docker-image:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      # 向 EVN 写入 CURRENT_VERSION 环境变量赋值版本号
      - run: |
          echo "CURRENT_VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_ENV
      # 可以获取到 CURRENT_VERSION 的值
      - run: echo "${{env.CURRENT_VERSION}}"
```

### 通过 commit 跳过任务

```yml
jobs:
  build:
    if: "!contains(github.event.head_commit.message, 'skip ci')"
```

### 获取是否存在 Tag

```yml
- run: echo "previous_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo '')" >> $GITHUB_ENV
- name: Generate changelog
  id: changelog
  uses: jaywcjlove/changelog-generator@main
  if: env.previous_tag
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
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./build
```

### 克隆带有 Submodule 的仓库

```yml
- name: Checkout
  uses: actions/checkout@v3
  with:
    path: main
    submodules: true
```

`submodules`：`true` 检出子模块或 `recursive` 递归检出子模块。

```yml
- name: Clone sub repository
  shell: bash
  run: |
    auth_header="$(git config --local --get http.https://github.com/.extraheader)"
    # git submodule sync --recursive
    # git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --remote --force --recursive --checkout ant.design
```

### 步骤依赖作业

使用 `jobs.<job_id>.needs` 识别在此作业运行之前必须成功完成的任何作业。它可以是一个字符串，也可以是字符串数组。 如果某个作业失败，则所有需要它的作业都会被跳过，除非这些作业使用让该作业继续的条件表达式。

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

在此示例中，`job1` 必须在 `job2` 开始之前成功完成，而 `job3` 要等待 `job1` 和 `job2` 完成。

此示例中的作业按顺序运行：

```
❶ job1
❷ job2
❸ job3
```

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    if: ${{ always() }}
    needs: [job1, job2]
```

在此示例中，`job3` 使用 `always()` 条件表达式，因此它始终在 `job1` 和 `job2` 完成后运行，不管它们是否成功。

### 步骤作业文件共享

Artifacts 是 GitHub Actions 为您提供持久文件并在运行完成后使用它们或在作业（文档）之间共享的一种方式。

要创建工件并使用它，您将需要不同的操作：上传和下载。
要上传文件或目录，您只需像这样使用它：

```yml
steps:
  - uses: actions/checkout@v2
  - run: mkdir -p path/to/artifact
  - run: echo hello > path/to/artifact/world.txt
  - uses: actions/upload-artifact@v2
    with:
      name: my-artifact
      path: path/to/artifact/world.txt
```

然后下载 `artifact` 以使用它：

```yml
steps:
  - uses: actions/checkout@v2
  - uses: actions/download-artifact@v2
    with:
      name: my-artifact
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
  uses: actions/setup-node@v2
  with:
    node-version: 18
```

```yml

jobs:
  test:
    # Containers must run in Linux based operating systems
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
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


## Contributors

As always, thanks to our amazing contributors!

<a href="https://github.com/jaywcjlove/github-actions/graphs/contributors">
  <img src="https://jaywcjlove.github.io/github-actions/CONTRIBUTORS.svg" />
</a>

Made with [action-contributors](https://github.com/jaywcjlove/github-action-contributors).

## License

Licensed under the MIT License.
