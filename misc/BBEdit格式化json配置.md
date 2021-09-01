# BBEdit格式化json配置



## 配置脚本

打开`~/Library/Application Support/BBEdit/Text Filters`	目录，

创建`jsonify.sh`

添加如下内容：

```sh
#!/bin/sh

python -m json.tool
```

重启BBEdit



## JSON格式化

选择一个未格式化的JSON

如：

```json
{
    "result_count": 3, 
    "results": [
        {
            "_href": "/ws.v1/lswitch/3ca2d5ef-6a0f-4392-9ec1-a6645234bc55", 
            "_schema": "/ws.v1/schema/LogicalSwitchConfig", 
            "type": "LogicalSwitchConfig"
        }, 
        {
            "_href": "/ws.v1/lswitch/81f51868-2142-48a8-93ff-ef612249e025", 
            "_schema": "/ws.v1/schema/LogicalSwitchConfig", 
            "type": "LogicalSwitchConfig"
        }, 
        {
            "_href": "/ws.v1/lswitch/9fed3467-dd74-421b-ab30-7bc9bfae6248", 
            "_schema": "/ws.v1/schema/LogicalSwitchConfig", 
            "type": "LogicalSwitchConfig"
        }
    ]
}
```

BBEdit中选择`Text>Apply Text Filters>jsonify `，格式化操作完成。