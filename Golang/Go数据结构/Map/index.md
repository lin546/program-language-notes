# Map

## **一、将 Map 转换为 Struct**
在Golang中，Struct 是字段的集合，Map 是键值对的集合。将 Map 转换为Struct的过程涉及创建一个新的Struct并将其字段设置为Map中的相应值。我们可以使用不同的方法来实现这一点。

### **1、使用 json.Unmarshal() 方法**
```plain
package main
import (
        "encoding/json"
        "fmt"
)
type MyStruct struct {
        Name    string 
json:"name"
        Age     int    
json:"age"
        Address string 
json:"address"
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

### **2、使用 mapstructure 库**
mapstructure 提供了一种使用 struct tags 将 map 转换为 struct 的方法。此外，它还提供了类型转换、默认值和验证等功能。

```plain
package main
import (
        "fmt"
        "github.com/mitchellh/mapstructure"
)
type MyStruct struct {
        Name    string
        Age     int
        Address map[string]string
}
func main() {
        // Define the map
        map1 := map[string]interface{}{
                "name": "Amit Kumar",
                "age":  30,
                "address": map[string]string{
                        "street": "316 Some Main Road",
                        "city":   "Crusasia",
                        "state":  "Karnataka",
                        "zip":    "803471",
                },
        }
        // Create a new struct to hold the data
        var data MyStruct
        // Use mapstructure to map the data from the map to the struct
        config := &mapstructure.DecoderConfig{
                ErrorUnused: true,
                Result:      &data,
        }
        decoder, err := mapstructure.NewDecoder(config)
        if err != nil {
                fmt.Println(err)
                return
        }
        if err := decoder.Decode(map1); err != nil {
                fmt.Println(err)
                return
        }
        // Print the struct data
        // {Name:Amit Kumar Age:30 Address:map[city:Crusasia state:Karnataka street:316 Some Main Road zip:803471]}
        fmt.Printf("%+v", data)
}
```

### **3、使用 for 循环语句**
使用简单的for循环遍历map元素并将其转换为struct：

```plain
package main
import "fmt"
type Identity struct {
        Name string
        Age  int
}
func main() {
        m := map[string]interface{}{
                "Name": "Amit Kumar",
                "Age":  30,
        }
        var p Identity
        for k, v := range m {
                switch k {
                case "Name":
                        p.Name = v.(string)
                case "Age":
                        p.Age = v.(int)
                }
        }
        // {Name:Amit Kumar Age:30}
        fmt.Printf("%+v\n", p)
}
```

### **4、使用 reflect package**
reflect 包提供了在运行时检查Go对象的类型和值的函数。所以我们也可以使用这个包实现。

```plain
package main
import (
        "fmt"
        "reflect"
)
type Identity struct {
        Name string
        Age  int
}
func convertMapToStruct(m map[string]interface{}, s interface{}) error {
        stValue := reflect.ValueOf(s).Elem()
        sType := stValue.Type()
        for i := 0; i < sType.NumField(); i++ {
                field := sType.Field(i)
                if value, ok := m[field.Name]; ok {
                        stValue.Field(i).Set(reflect.ValueOf(value))
                }
        }
        return nil
}
func main() {
        m := map[string]interface{}{
                "Name": "Amit Kumar",
                "Age":  30,
        }
        var p Identity
        err := convertMapToStruct(m, &p)
        if err != nil {
                fmt.Println(err)
                return
        }
        fmt.Printf("%+v\n", p) // {Name:Amit Kumar Age:30}
}
```

在这个例子中，我们定义了一个以map和struct指针为参数的函数转换器MapTostruct。在函数内部，我们使用reflect包分别获取stValue和sType内部结构的值和类型。然后我们使用for循环遍历struct字段，并检查Map中是否存在字段名称。如果存在，我们使用反射包将struct字段值设置为相应的map值。



> 更新: 2024-10-03 22:04:27  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/kc92hyws38nlxpde>