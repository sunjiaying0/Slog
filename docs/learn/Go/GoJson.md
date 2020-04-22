# Go的json解析：Marshal与Unmarshal

json(Javascript Object Nanotation)是一种数据交换格式，常用于前后端数据传输，对于前后端分离的项目来讲 是极其重要的，任意一端将数据换成json字符串，另一段再将该字符串解析成相应的数据结构，如string类型，strcut对象等。<br>

一个JSON数组是一个有序的值序列，写在一个方括号中并以逗号分隔；一个JSON数组可以用于编码Go语言的数组和slice。一个JSON对象是一个字符串到值的映射，写成以系列的name:value对形式，用花括号包含并以逗号分隔；JSON的对象类型可以用于编码Go语言的map类型（key类型是字符串）和结构体
```
boolean         true
number          -273.15
string          "She said \"Hello, BF\""
array           ["gold", "silver", "bronze"]
object          {"year": 1980,
                 "event": "archery",
                 "medals": ["gold", "silver", "bronze"]}
```

`Json Marshal(编组)：将数据编码成json字符串`<br>
`Json Unmarshal(解码)：将json字符串解码到相应的数据结构`
```go
type Stu struct {
	Name  string `json:"name"`
	Age   int
	HIgh  bool
	sex   string
	Class *Class `json:"class"`
}

type Class struct {
	Name  string
	Grade int
}

type Movie struct {
	Title  string
	Year   int  `json:"released"`
	Color  bool `json:"color,omitempty"`
	Actors []string
}

var movies = []Movie{
	{Title: "Casablanca", Year: 1942, Color: false,
		Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
	{Title: "Cool Hand Luke", Year: 1967, Color: true,
		Actors: []string{"Paul Newman"}},
	{Title: "Bullitt", Year: 1968, Color: true,
		Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
	// ...
}

func main() {
	//实例化一个数据结构，用于生成json字符串
	stu := Stu{
		Name: "张三",
		Age:  18,
		HIgh: true,
		sex:  "男",
	}

	//指针变量
	cla := new(Class)
	cla.Name = "1班"
	cla.Grade = 3
	stu.Class=cla

	//Marshal失败时err!=nil
	jsonStu, err := json.Marshal(stu)
	if err != nil {
		fmt.Printf("生成json字符串错误%s",err)
	}
	jsonStu1, err := json.Marshal(movies)
	if err != nil {
		fmt.Printf("生成json字符串错误:%s",err)
	}

	//jsonStu是[]byte类型，转化成string类型便于查看
	fmt.Println(string(jsonStu))
	fmt.Println(string(jsonStu1))
	var student Stu
    err := json.Unmarshal([]byte(jsonStu),&student)
    if err != nil {
		fmt.Printf("解析json字符串错误:%s",err)
	}
	fmt.Println(student)
}

```

`json.Marshal()`知识点：<br>
* 只要是可导出成员（变量首字母大写），都可以转成json。因成员变量sex是不可导出的，故无法转成json

* 如果变量打上了json标签，如Name旁边的 `json:"name"` ，那么转化成的json key就用该标签“name”，否则取变量名作为key，如“Age”，“HIgh”

* bool类型也是可以直接转换为json的value值。Channel， complex 以及函数不能被编码json字符串 当然，循环的数据结构也不行，它会导致marshal陷入死循环

* 指针变量，编码时自动转换为它所指向的值，如cla变量
（当然，不传指针，Stu struct的成员Class如果换成Class struct类型，效果也是一模一样的 只不过指针更快，且能节省内存空间）

* json编码成字符串后就是纯粹的字符串了

> 上面的成员变量都是已知的类型，只能接收指定的类型，比如string类型的Name只能赋值string类型的数据<br>
> 但有时为了通用性，或使代码简洁，我们希望有一种类型可以接受各种类型的数据，并进行json编码。这就用到了interface{}类型<br>
> interface{}类型其实是个空接口，即没有方法的接口。go的每一种类型都实现了该接口。因此，任何其他类型的数据都可以赋值给interface{}类型

`json.Unmarshal()`知识点：<br>
* json字符串解析时，需要一个“接收体”接受解析后的数据，且Unmarshal时接收体必须传递指针。否则解析虽不报错，但数据无法赋值到接受体中。如这里用的是StuRead{}接收。

* 解析时，接收体可自行定义。json串中的key自动在接收体中寻找匹配的项进行赋值。匹配规则是：<br>
    (1) 先查找与key一样的json标签，找到则赋值给该标签对应的变量(如Name)<br>
    (2) 没有json标签的，就从上往下依次查找变量名与key一样的变量，如Age或者变量名忽略大小写后与key一样的变量。如HIgh，Class,第一个匹配的就赋值，后面就算有匹配的也忽略(前提是该变量必需是可导出的，即首字母大写)<br>
* 不可导出的变量无法被解析（如sex变量，虽然json串中有key为sex的k-v 解析后其值仍为nil,即空值）

* 当接收体中存在json串中匹配不了的项时，解析会自动忽略该项，该项仍保留原值。如变量Test，保留空值nil
