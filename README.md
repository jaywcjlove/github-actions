# github-actions


## [Github 上下文](https://help.github.com/cn/actions/reference/context-and-expression-syntax-for-github-actions)

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