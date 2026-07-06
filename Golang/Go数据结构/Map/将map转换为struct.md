# 将 map 转换为 struct

### 1、使用 <font style="color:rgb(199, 37, 78);background-color:rgb(249, 242, 244);">json.Unmarshal() </font>方法
```go
package main

import (
	"encoding/json"
	"fmt"
)

type MyStruct struct {
	Name    string `json:"name"`
	Age     int    `json:"age"`
	Address string `json:"address"`
}

func main() {
	// Define the map
	map1 := map[string]interface{}{
		"name":    "Amit Kumar",
		"age":     30,
		"address": "316 Some Main Road, Cruasia",
	}

	// Convert the map to JSON
	jsonData, _ := json.Marshal(map1)

	// Convert the JSON to a struct
	var structData MyStruct
	json.Unmarshal(jsonData, &structData)

	fmt.Println(structData) // {Amit Kumar 30 316 Some Main Road, Cruasia}
}
```





## 参考：
+ [Convert map to struct in Golang [4 Methods] | GoLinuxCloud](https://www.golinuxcloud.com/go-map-to-struct/)



> 更新: 2023-08-27 21:15:14  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/li99q53swc1loih4>