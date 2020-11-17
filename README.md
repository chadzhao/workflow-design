# 工作流设计

    基本思想

- 表单和流程节点使用Json字义
- 表单和流程节点使用版本，一旦有流程建立则固定使用该版本
- 工作流数据根据字段类型拆开保存，便于搜索、排序及动态化
- 表单打印使用HTML
- 表单导出Excel与表单布局不一致，以方便导入

## 表单设计器
- 逐行定义表单数据，可定义每行高度
- 宽度以A4纸宽度为基准，按百分比分别设置每列宽度
- 字段名和填空都可设置跨列数
- 使用Json序列化保存表单设计
- Json数据定义

```json
  {
      'cols' : [10,20,30,20,20], // 每列宽度定义（单位：百分比）
      'maxFieldId' : 1, // 当前已分配的最大字段Id（防止重复使用）
      'rows' : [ // 表单每行定义 
        {
            'height' : 20, // 行高(单位：像素)
            'fields' : [ 
                { // 该行字段
                    'id' : 1, // 每个字段都有唯一的表单内id（使用内部自增即可）
                    'colSpan' : 1, // 所占列数
                    'name' : '项目名称', // 字段名称
                    'type' : 'text', // 字段类型(text/number/select/file)
                    'style' : { // 额外的样式（比如文字过多可以把字体设置小）
                        'font-size' : '12px'
                    }
                }
            ]
        }
          
      ],
      'maxFlowId' : 1, // 当前已分配的最大流程节点Id（防止重复使用）
      'flows' : [ // 工作流流程节点定义
        {
            'id' : 1, // 流程节点Id，内部自增（Id为1的永远是最初的起点）
            'name' : '建立表单', // 流程节点名称
            'fields' : {
                'mode' : 'AllVisible', // AllVisible|AllEditable|Custom
                'editable' : [1,2], // Custom模式下，指定可编辑的字段
                'visible' : [3,4] // Custom模式下，指定可见（不可编辑）的字段
            },
            'next' : { // 下一节点信息
                'flowId' : 2, // 下一节点Id
            }
        }
      ]
  }
```

## 工作流设计
- 


## 数据库表格

#### 工作流模板表 utWfTemplate

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| Name | string | 工作流名称 |
| FormConfig | string | 表单Json数据 |

#### 表单版本表 utWfFormVersion

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| WfTemplateId | int | 对应工作流模板Id |
| FormConfig | string | 表单Json数据 |

#### 工作流记录表 utWfRecord

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| WfTemplateId | int | 对应工作流模板Id |
| Version | int | 对应表单版本Id |
| AcceptorId | int | 当前负责人Id |

#### 工作流表单数据表 utWfForm+字段类型  

    每种字段类型建立一个表，如 `utWfFormString` `utWfFormInt` `utWfFormDate`

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | string | 格式：工作流记录Id_字段Id |
| WfRecordId | int | 工作流记录Id |
| FieldId | int | 字段Id |
| Value | string/int/etc | 根据字段类型设置不同类型 |

#### 工作流节点表 utWfItem

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| FlowId | int | 流程节点内部Id |
| Name | string | 节点名称 |
| AcceptorId | int | 当前节点负责人Id |
| IsAccepted | bool? | 是否通过 |
| Remark | string | 通过或未通过的批注 |
| FinishTime | date? | 完成时间 |
| CreateTime | date | 创建时间 |