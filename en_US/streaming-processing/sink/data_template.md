# Data template

After users perform data analysis and processing through NeuronEX, they can use various actions (Sink) to send data analysis results to different systems. For the same analysis results, different actions (Sink) may not necessarily require the same format.

For example, in a certain scenario, when it is found that the temperature of a device is too high, a request needs to be sent to a REST service in the cloud, and a control command needs to be sent to the device locally through the MQTT protocol. The data formats required for the two may not be the same. They are different. Therefore, the results from the analysis need to be "secondary processed" before targeted data can be sent to different targets. This article will introduce how to use the `Data Template` in Sink to implement "secondary processing" of analysis results.

Log in to NeuronEX and click **Data Processing** -> **Rules**. On the **Rules** tab, click **Create Rule** -> **Add Action** and select a specific action (Sink) to use the data template function.

<img src="../_assets/data_template1.png" alt="data_template1" style="zoom:100%;" />

## Golang template introduction

The Golang template applies a piece of logic to the data, and then formats and outputs the data according to the logic specified by the user. The common usage scenario of the Golang template is in web development. For example, after converting and controlling a data structure in Golang, it is converted to HTML tags and output to the browser. NeuronEX uses [Golang template](https://golang.org/pkg/text/template/) to implement "secondary processing" of the analysis results. Please refer to the following introduction from Golang.

> Templates are executed by applying them to a data structure. Annotations in the template refer to elements of the data structure (typically a field of a struct or a key in a map) to control execution and derive values to be displayed. Execution of the template walks the structure and sets the cursor, represented by a period '.' and called "dot", to the value at the current location in the structure as execution proceeds.
>
> The input text for a template is UTF-8-encoded text in any format. "Actions"--data evaluations or control
> structures--are delimited by <span v-pre>"{{" and "}}"</span>; all text outside actions is copied to the output
> unchanged. Except for raw strings, actions may not span newlines, although comments can.

## Simple Template

If sendSingle is true, the data template will execute against a record; Otherwise, it will execute against the whole array of records. Typical data templates are:

For example, we have the sink input as

```go
[]map[string]interface{}{{
    "ab" : "hello1",
},{
    "ab" : "hello2",
}}
```

In sendSingle=true mode:

- Print out the whole record

```text
"dataTemplate": "{\"content\":{{json .}}}",
```

- Print out the ab field

```text
"dataTemplate": "{\"content\":{{.ab}}}",
```

if the ab field is a string, add the quotes

```text
"dataTemplate": "{\"content\":\"{{.ab}}\"}",
```

In sendSingle=false mode:

- Print out the whole record array

```text
"dataTemplate": "{\"content\":{{json .}}}",
```

- Print out the first record

```text
"dataTemplate": "{\"content\":{{json (index . 0)}}}",
```

- Print out the field ab of the first record

```text
"dataTemplate": "{\"content\":{{index . 0 \"ab\"}}}",
```

- Print out field ab of each record in the array to html format

```text
"dataTemplate": "<div>results</div><ul>{{range .}}<li>{{.ab}}</li>{{end}}</ul>",
```

Actions could be customized to support different kinds of outputs, see [extension](../extension.md) for more detailed info.

## Functions supported in template

With the help of template functions, users can do a lot of transformation including formation, simple mathematics, encoding etc. The supported functions in NeuronEX template includes:

1. Go built-in [template functions](https://golang.org/pkg/text/template/#hdr-Functions).
2. An abundant extended function set from [sprig library](http://masterminds.github.io/sprig/).
3. NeuronEX extended functions.


## Actions

The Golang  template provides some [built-in actions](https://golang.org/pkg/text/template/#hdr-Actions) which allows users to write various control statements to extract content. For example,

- Output different contents according to judgment conditions

```text
{{if pipeline}} T1 {{else}} T0 {{end}}
```

- Iterate through  the data and process it

```text
{{range pipeline}} T1 {{else}} T0 {{end}}
```

Readers can see that actions are delimited by <code v-pre>{{}}</code>. During the use of NeuronEX data templates, the
output is generally in JSON format, and the JSON format is delimited by `{}`. Therefore, if the readers are not familiar
with it, they will find it difficult to understand the functions of NeuronEX's data templates. For example, in the
following example,

```text
{{if pipeline}} {"field1": true} {{else}}  {"field1": false} {{end}}
```

The meaning of the above expression is as follows (please note the delimiter of action and the delimiter of JSON):

- If the condition pipeline is met, the JSON string `{"field1": true}` is output
- Otherwise, the JSON string `{"field1": false}` is output

## NeuronEX sink data format

The Golang template can be applied to various data structures, such as maps, slices, channels, etc., and the data type obtained by the data template in NeuronEX's sink is fixed, which is a data type that contains Golang `map` slices. It is shown as follows.

```go
[]map[string]interface{}
```

## Send slice data by piece

The data flowing into the sink is a  data structure of `map[string]interface{}` slice. However, when the user sends data to the target sink, it may need a single piece of data instead of all the data. 

```json
[
  {"device_id":"1","t_av":36.25,"t_count":4,"t_max":80,"t_min":10},
  {"device_id":"2","t_av":27,"t_count":4,"t_max":45,"t_min":12}
]
```

::: v-pre
When sending to the sink, each piece of data is sent separately. First, you need to set the `sendSingle` of the sink to `true`, and then use the data template: `{{json .}}`. The complete configuration is as follows, and the user can copy it to the end of a sink configuration.
:::

```json
 ...
 "sendSingle": true,
 "dataTemplate": "{{toJson .}}"
```

- After setting `sendSingle` to `true`, NeuronEX traverses the `[]map[string]interface{}` data type that has been passed to the sink. For each data in the traversal process, the user-specified data template will be applied.
- `toJson` is a function provided by NeuronEX (users can refer to [Template Functions in NeuronEX](#functions-supported-in-template) for more information of NeuronEX extensions), which can convert incoming parameters into JSON string output. For each piece of traversed data, the content in the map is converted to a JSON string

Golang also provides some built-in functions. Users can refer to [More Golang Built-in Functions](https://golang.org/pkg/text/template/#hdr-Functions) for more function information.

## Data content conversion

Still for the above example, you need to do some conversion on the returned `t_av` (average temperature). The basic requirement of the conversion is to add different description text according to different average temperatures for processing in the target sink. The rules are as follows,

- When the temperature is less than 30, the description field is "Current temperature is`$t_av`,  it's normal."
- When the temperature is greater than 30, the description field is "Current temperature is`$t_av`, it's high."

Assuming that the target sink still needs JSON data, the content of the data template is as follows:

```json
...
"dataTemplate": "{\"device_id\": {{.device_id}}, \"description\": \"{{if lt .t_av 30.0}}Current temperature is {{.t_av}}, it's normal.\"{{else if ge .t_av 30.0}}Current temperature is {{.t_av}}, it's high.\"{{end}}}"
"sendSingle": true,
```

::: v-pre
In the above data template, the built-in actions of <code v-pre>{{if pipeline}} T1 {{else if pipeline}} T0
{{end}}</code> are used, which looks more complicated. We can do a little adjustment, remove the escape and add
abbreviation. The typesetting afterwards is as follows (note: when generating NeuronEX rules, the following optimized
typesetting rules cannot be passed in).
:::

```text
{"device_id": {{.device_id}}, "description": "
  {{if lt .t_av 30.0}}
    Current temperature is {{.t_av}}, it's normal."
  {{else if ge .t_av 30.0}}
    Current temperature is {{.t_av}}, it's high."
  {{end}}
}
```

Use Golang's built-in binary comparison function:

- `lt`：less than
- `ge`：greater or equal to

It is worth noting that in the `lt` and `ge` functions, the type of the second parameter value should be consistent with the actual data type of the data in the map, otherwise an error will occur. As in the above example, when the temperature is greater than `30`, because the type of actual average number in map is float, the value of the second parameter needs to be passed into `30.0`, not `30`.

In addition, the template is still applied to each record in the slice, so you still need to set the `sendSingle` attribute to `true`. Finally, the content generated by the data template for the above data is as follows,

```json
{"device_id": 1, "description": "Current temperature is 36.25, it's high."}
{"device_id": 2, "description": "Current temperature is 27, it's normal."}
```

## Data traversal

By setting the `sendSingle` property of the sink to `true`, the slice data passed to the sink can be traversed. Here, we will introduce some more complex examples. For example, for the result of the sink which contains data of the nested array type, how can it realize the traversal by the traversal function provided in the data template.

Assuming that the data content flowing into the sink is as follows:

```json
{"device_id":"1",
 "values": [
  {"temperature": 10.5},
  {"temperature": 20.3},
  {"temperature": 30.3}
 ]
}
```

The requirement is:

- When a value of `temperature` in the "values" array is found to be less than or equal to `25`, add an attribute named `description` and set its value to `fine`.
- When a value of `temperature` in the "values" array is found to be greater than `25`, add an attribute named `description` and set its value to `high`.

```json
"sendSingle": true,
"dataTemplate": "{{$len := len .values}} {{$loopsize := add $len -1}} {\"device_id\": \"{{.device_id}}\", \"description\": [{{range $index, $ele := .values}} {{if le .temperature 25.0}}\"fine\"{{else if gt .temperature 25.0}}\"high\"{{end}} {{if eq $loopsize $index}}]{{else}},{{end}}{{end}}}"
```

The data template is relatively complicated, which is explained below:

::: v-pre

- `{{$len := len .values}} {{$loopsize := add $len -1}}`, this section executes two expressions. For the first one, `len` function gets the length of `values` in the data. For the second one, `add` decrements its value by 1 and assigns it to the variable `loopsize`. At present, since the operation of directly decrementing the value by 1 is not supported by  the Golang expression, `add` is a function extended by NeuronEX to achieve this function.
:::

::: v-pre

- `{\"device_id\": \"{{.device_id}}\", \"description\": }` This piece of template is applied to the sample data, and a JSON string `{"device_id": "1", "description": }` is generated.
:::

::: v-pre

- `{{range $index, $ele := .values}} {{if le .temperature 25.0}}\"fine\"{{else if gt .temperature 25.0}}\"high\"{{end}} {{if eq $loopsize $index}}]{{else}},{{end}}{{end}}` ,this section of the template looks relatively complicated. However, if we adjust it, remove the escape and add indentation, the  typesetting is as follows which may be clearer (note: when generating the NeuronEX rules, the following optimized typesetting rules cannot be passed in).
:::

  ```text
  {{range $index, $ele := .values}}
    {{if le .temperature 25.0}}
      "fine"
    {{else if gt .temperature 25.0}}
      "high"
    {{end}}
    {{if eq $loopsize $index}}
      ]
    {{else}}
      ,
    {{end}}
  {{end}}
  ```

  The first condition judges whether to generate `fine` or `high`; the second condition judges whether to generate `,` to separate the array or `]` at the end of the array.

In addition, the template is still applied to each record in the slice. Therefore, we still need to set the `sendSingle` attribute to `true`. Finally, the content generated by the data template for the above data is as follows:

```json
  {"device_id": "1", "description": [ "fine" , "fine" , "high" ]}
```

## AI-assisted generation

NeuronEX's template syntax is the same as the Go language, so it is easy to generate data templates with AI assistance. For example, We can use the following prompt to send to a large language model to assist in generating data templates:

```sh
Use Golang's text/template and the sprig library to convert the data
{
    "timestamp": 1650006388943,
    "node": "modbus",
    "group": "grp",
    "values": {
        "tag1": 123,
        "tag2": 234
    }
}
into
{
  "timestamp": 1650006388943,
  "node": "modbus",
  "group": "grp",
  "tags": [{
        "name": "tag1",
        "value": 123
      },{
        "name": "tag2",
        "value": 234
      }
  ]
}
```

The resulting template is approximately as follows:

```json
{"timestamp": {{.timestamp}},"node": "{{ .node }}","group": "{{ .group }}","tags": [{{ range $key, $value := .values }}{{ if $key }},{ "name": "{{ $key }}", "value": {{ $value }} }{{ else }}{ "name": "{{ $key }}", "value": {{ $value }} }{{ end }}{{ end }}]}
```
Using the rule debugging feature, it was tested and found that the conversion works correctly:

![alt text](_assets/date_tp_1.png)

![alt text](_assets/date_tp_2.png)

![alt text](_assets/date_tp_3.png)

By sending the input and output templates to a large language model, even those unfamiliar with Golang template syntax can easily achieve data conversion.

## Summary

The data template function provided by NeuronEX can realize the secondary processing of the analysis results to meet the needs of different sink targets. However, readers can also see that due to the limitations of the Golang template, it is awkward to implement more complex data conversion. We hope that the Golang template function can be made more powerful and flexible in the future, which can support more complex requirements. At present, it is recommended that users can implement some simpler data conversion through data templates. If the user needs to perform more complicated processing on the data and extends the sink by himself, it can be directly processed in the sink implementation.

