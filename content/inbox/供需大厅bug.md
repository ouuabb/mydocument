统计返回的字段待渲染waitResponseNum

| 状态码 | 原名称    | 新业务名称 | 统计数据状态码         |
| --- | ------ | ----- | --------------- |
| “”  | 全部     | 无     | allNum          |
| 1   | 发布     | 待响应   | waitResponseNum |
| 2   | 待付款    | 待选择   | waitSelectNum   |
| 3   | 支付完成   | 待服务   | waitServiceNum  |
| 4   | 供方开始服务 | 服务中   | serviceNum      |
| 5   | 供方结束服务 | 待确认   | completeNum     |
| 6   | 需方确认   | 已确认   | confirmedNum    |

发布完成之后状态是1，选择后状态变更为2，不能再选择其他响应
