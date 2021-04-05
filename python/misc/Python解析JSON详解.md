# Python解析JSON详解

## JSON 函数

使用 JSON 函数需要导入 json 库：import json。

函数  描述

json.dumps  将 Python 对象编码成 JSON 字符串

json.loads  将已编码的 JSON 字符串解码为 Python 对象

### json.dumps

语法

json.dumps(obj, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, encoding="utf-8", default=None, sort_keys=False, **kw)

实例

以下实例将数组编码为 JSON 格式数据：

```python
#!/usr/bin/python
import json
data = {'number': 6, 'name': 'Pythontab'}
jsonData = json.dumps(data)
print jsonData
```



以上代码执行结果为：

```
{"number": 6, "name": "Pythontab"}
```



注意： 大家可能发现，执行上述转换以后，数据并没有发生变化，这里要说一下： 在json中双引号才是标注的字符串分割符号，单引号不标准。

使用参数让 JSON 数据排序并格式化输出：

```
#!/usr/bin/python
import json
data = {'number': 6, 'name': 'Pythontab'}
jsonData = json.dumps(data, sort_keys=True, indent=4, separators=(',', ': '))
print jsonData
```



输出结果

```
{
    "name": "Pythontab",
    "number": 6
}
```



python 原始类型向 json 类型的转化对照表：

| Python           | JSON   |
| ---------------- | ------ |
| dict             | object |
| list, tuple      | array  |
| str, unicode     | string |
| int, long, float | number |
| True             | true   |
| False            | false  |
| None             | null   |

### json.loads

json.loads 用于解码 JSON 数据。该函数返回 Python 字段的数据类型。

语法

json.loads(s[, encoding[, cls[, object_hook[, parse_float[, parse_int[, parse_constant[, object_pairs_hook[, **kw]]]]]]]])

实例

以下实例展示了Python 如何解码 JSON 对象：

```
#!/usr/bin/python
import json
jsonData = '{"number": 6, "name": "Pythontab"}'
str = json.loads(jsonData)
print str
```



以上代码执行结果为：

```
{u'number': 6, u'name': u'Pythontab'}
```



json 类型转换到 python 的类型对照表：

| JSON          | Python    |
| ------------- | --------- |
| object        | dict      |
| array         | list      |
| string        | unicode   |
| number (int)  | int, long |
| number (real) | float     |
| true          | True      |
| false         | False     |
| null          | None      |

### 使用第三方库：Demjson

Demjson 是 python 的第三方模块库，可用于编码和解码 JSON 数据，包含了 JSONLint 的格式化及校验功能。

Github 地址：https://github.com/dmeranda/demjson

#### 环境配置

在使用 Demjson 编码或解码 JSON 数据前，我们需要先安装 Demjson 模块。

方法1：源码安装

```
$ tar -xvzf demjson-2.2.4.tar.gz

$ cd demjson-2.2.4

$ python setup.py install
```

方法2：直接使用pip安装

```
pip install Demjson
```

#### JSON 函数

函数  描述

encode  将 Python 对象编码成 JSON 字符串

decode  可以使用 demjson.decode() 函数解码 JSON 数据。该函数返回 Python 字段的数据类型。

encode语法

demjson.encode(self, obj, nest_level=0)

decode语法

demjson.decode(self, txt)