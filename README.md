# 工作流设计

    基本思想

- 表单和流程节点使用Json字义
- 表单和流程节点使用版本，一旦有流程建立则固定使用该版本
- 工作流数据直接Json保存，MySql支持Json查询，数据量不大的时候性能影响很小
    - 工作流查询已按模板过滤一次，数据量大大减少
    - 后台OA系统，使用动态字段查询频率本来就不会高。
    - 用户能容忍查询稍微慢一点
    - 工作流数据量远达不到大量的程度（1万以上才算）
- 表单打印使用HTML
- 表单导出Excel与表单布局不一致，以方便导入

## 表单设计器
- 逐行定义表单数据，可定义每行高度
- 宽度以A4纸宽度为基准，按百分比分别设置每列宽度（或使用像素也行）
- 字段名和填空都可设置跨列数
- 使用Json序列化保存表单设计
- Json数据定义

```javascript
var config = 
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

#### 表单设计器渲染技巧 `VUE/Angular`

- 逐行循环显示表单元素
- 对于每行，遍历该行元素，根据元素的colSpan计算出Left和Width值，定位显示出来
- 根据元素类型动态显示相应组件
- 当选中某元素时，标记当前已选元素 selectedField=e; 右侧的设计器双向绑定其属性直接编辑
- 新增元素时maxFieldId自动加1，删除时不回减（防止重复使用）
- 每行最开始提供选择整行功能，进而可以设置行高
- 提供A4纸尺寸参考，防止超出布局（打印优化）
- 以上方案也可用在生成Excel中

## 工作流设计
- 


## 数据库表

#### 工作流模板表 `utWfTemplate`

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| Name | string | 工作流名称 |
| FormConfig | string | 表单Json数据 |
| Version | int | 当前版本 |
| Fields | string | 此模板拥有的所有动态字段：包括历史记录 |

```javascript
    // fields格式定义（同Id的Name取最新名称，标记最新版本是否还在使用）
    var json = [
        { id : 1, type : text, name : '字段名称', isInUse : true },
        { id : 2, type : int, name : '字段名称', isInUse : true },
        { id : 3, type : float, name : '字段名称', isInUse : false },
    ];
```

#### 表单版本表 `utWfFormVersion`
    只有修改流程、增删字段才会生成新版本，否则直接视为最新版（模板和当前版本一起更新）
| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| WfTemplateId | int | 对应工作流模板Id |
| FormConfig | string | 表单Json数据 |

#### 工作流记录表 `utWfRecord`

| 字段 | 类型 | 说明 |
| ---- | ---- | ---- |
| Id | int | 自增 |
| WfTemplateId | int | 对应工作流模板Id |
| Version | int | 对应表单版本Id |
| AcceptorId | int | 当前负责人Id |
| FormData | string | Json形式保存所有Form值，格式见下方 |

```javascript
// FormData Json数据格式
var json = {
    's1' : '字符串使用s+字段Id形式',
    'i2' : '整形使用i+字段Id形式',
    'f3' : '浮点使用f+字段Id形式',
    'd4' : '时间使用d+字段Id形式',
};
```

#### 工作流节点表 `utWfItem`

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

## 常规查询

#### 动态字段匹配查询
    比如查找模板为2，字段id=1，类型为string，关键字为keyword的记录.
```sql
    -- 先查询工作流记录
    select * from utWfRecord where wfTemplateId=2 and JSON_EXTRACT(FormData, '$.s1') like '%keyword%'

    -- 再根据Version加载表单定义，将FormData还原
```