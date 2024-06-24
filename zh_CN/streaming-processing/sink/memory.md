# 内存 Sink

<span style="background:green;color:white">updatable</span>

该动作用于将结果刷新到内存中的主题中，方便 [内存源](../memory.md) 使用。 该主题类似于 pubsub 主题，例如 mqtt，因此可能有多个内存目标发布到同一主题，也可能有多个内存源订阅同一主题。 内存动作的典型用途是形成[规则流水线](../rule_pipeline.md)。

如希望使用 Memory Sink 连接器，点击 **数据处理** -> **规则** -> **新建规则**，在 **动作** 区域，点击**添加**，**Sink** 选择 **memory**。

## Sink 配置

在规则页面，点击**添加动作**，进行如下设置：

- **主题**：主题，例如 analysis/result
- **动作字段**：指定插入或更新动作，默认为插入操作
- **Key字段**：指定代表消息 key 的字段，如配置**动作字段**，则也应配置 **Key 字段**。
- **是否忽略输出**：默认为 False。
- **将结果数据按条发送**：默认为 True。
- **流格式**：默认为 `json`。
- **数据模版**：Golang 模板，用于指定输出数据格式。如不指定数据模板，则将数据作为原始输入。关于数据模版的详细介绍，见 [数据模版](./data_template.md)

完成设置后，可点击**测试连接**确认连接情况。最后点击**提交**，完成设置。

## 示例

内存动作配置示例：

```json
{
  "memory": {
    "topic": "devices/result"
  }
}
```

动态主题示例：

```json
{
  "memory": {
    "topic": "{{.topic}}"
  }
}
```

## 数据模板

::: v-pre
内存动作和内存源之间的数据传输采用内部格式，不经过编解码以提高效率。因此，内存动作的格式相关配置项，除了数据模板之外都会被忽略。内存动作可支持数据模板对结果格式进行变化，但是数据模板的结果必须为 JSON 字符串的 object 形式，例如 `"{\"key\":\"{{.key}}\"}"`。数组形式的 JSON 字符串或者非 JSON 字符串都不支持。
:::

## 更新

内存动作支持[更新](./sink.md)。可用于更新订阅了与 sink 相同的主题的查询表。一个典型的用法是创建一个规则，使用可更新的 sink 来累积更新内存表。在下面的例子中，来自流alertStream的数据将更新内存主题`alertVal`。更新动作是由流入的数据中的 `action` 字段指定的。

```json
{
  "id": "ruleUpdateAlert",
  "sql":"SELECT * FROM alertStream",
  "actions":[
    {
      "memory": {
        "keyField": "id",
        "rowkindField": "action",
        "topic": "alertVal",
        "sendSingle": true
      }
    }
  ]
}
```