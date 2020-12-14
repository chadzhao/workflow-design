## 需要修改的表

#### 工作流模板表 TbWfTemplate

> 以下字段大部分来自 `新建-基础设置` 中

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| Remark | 表单说明 | varchar | 200 |  |
| AuthCreaters | 可发起人 | text | | Json数组，保存发起人Id列表，如：[1,4,6] |
| ShenpiOptions | 审批去重设置 | int | | 对应后台是枚举 |
| ApproveForShenpiCreate | 审批人发人是同一人自动通过 | bool | |
| ShenpiDesTips | 审批意见填写提示 | varchar | 100 | |
| ShenpiDesReqried | 审批意见必填 | bool | | |
| ShenpiForOthers | 允许代他人提交 | bool | | |
| HandSignRerquied | 是否必须手写签名 | bool | | |
| IsPublished | 是否已发布 | bool | | 只有已发布的才允许创建工作流 |

#### 工作流记录表 TbWfRecord

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| IsPublished | 是否已发布 | bool | | 在创建工作流时，如果还没确定，可以暂存为草稿，此时状态为未发布 |


#### 工作流节点表 TbWfItem

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| AgentId | 代理人Id | int | | 如果是代理人操作，此处保存代理人的Id |


#### 工作流处理表 TbWfUser

> 用来标识谁处理过此工作流，包括创建，只要操作过工作流就保存

> `**需要问一下由代理人处理的是否也出现在被代理人的处理过的工作流中**`

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| PkCode | Id | varchar | 50| 工作流记录Id_处理人Id |
| fkWfRecordId | 工作流记录Id | int | | 工作流记录Id |
| fkUserId | 用户Id | int | | 用户Id |
| tCreate | 创建时间 | datetime | | |

#### 工作流抄送表 TbWfRefer

> 用来标识工作流抄送

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| PkCode | Id | varchar | 50| 工作流记录Id_被抄送人Id |
| fkWfRecordId | 工作流记录Id | int | | 工作流记录Id |
| fkUserId | 用户Id | int | | 用户Id |
| fkOperatorId | 发起人Id | int | 谁发的抄送 |
| tCreate | 创建时间 | datetime | | |

#### 工作流附件表 TbWfAttachment

> 用来保存工作流表单中上传的文件（即便已经在工作流Form中以Json方式保存过文件路径），方便后面查找。
> 同时记录了此文件在表单中的字段名称，后期可以根据字段列出这些文件来（更容易查找）

| 字段名称 | 字段意思 | 类型 | 大小 | 备注 |
| ----- | ----- | ----- | ----- | ----- |
| PkCode | Id | int | | 自增 |
| fkWfRecordId | 工作流记录Id | int | | 工作流记录Id |
| fkOperatorId | 操作人 | int | | 操作人 |
| FilePath | 文件路径 | varchar | | 文件保存路径，如：doc/2020/12/15/1213242425353.doc |
| FieldId | 内部字段Id | int | | 来自工作流模板中定义此文件的字段Id |
| FieldName | 内部字段名称 | varchar | | 来自工作流模板中定义此文件的字段名称 |
| tCreate | 创建时间 | datetime | | |
