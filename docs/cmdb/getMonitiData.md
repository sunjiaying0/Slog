# 完成该需求过程中的常见代码块及常用知识点

## 转发请求时使用到的常见代码
```go
// 发起get请求
func callMonitorHostDataByIps(ips_lis []string,token string, url string) (map[string]interface{}, error) {
	var resdata map[string]interface{}
	request, err := http.NewRequest("GET", url,nil)
	if err != nil {
		// handle error
		blog.Errorf("http failed, error:%s", err)
	}
	request.Header.Set(common.BKHTTPAUTHORIZATION, token)

	client := http.Client{}
	res, err := client.Do(request) //Do 方法发送请求，返回 HTTP 回复
	defer res.Body.Close()
	body, err := ioutil.ReadAll(res.Body)
	if err != nil  {
		blog.Errorf("[snap] http get host zabbix info failed, err: %v, getURL:%v", err, url)
		return nil,err
	}

	json.Unmarshal(body, &resdata)
	if resdata["status"] != float64(0){
		err = fmt.Errorf("%s", resdata["msg"])
		return nil, err
	}
	return resdata, nil
}
```

```go
// 获取请求用户的token：
u := req.Request.Header.Get(common.BKHTTPAUTHORIZATION)
```

```go
// 该方法虽然没有用到，但个人觉得挺有用的
type JsonResut struct {
    status float64 `json:"status"`
    msg string `json:"msg"`
    data map[string]interface{} `json:"data"`
}
var resdata1 JsonResut
json.Unmarshal([]byte(body), &resdata)
```
```go
// 获取url内参数
func (s *Service) HostSnapMountedDiskIopsByMoni(req *restful.Request, resp *restful.Response) {
    // start_time与end_time是参数传递
    // 例如：http://127.0.0.1:8090/api/v3/hosts/snapshot/18/?start_time=1587449571.918&end_time=1587453171.918 问号后的参数数据
	start_time := req.QueryParameter("start_time")
    end_time := req.QueryParameter("end_time")
    // hostID 是url的一部分
    // 例如：http://127.0.0.1:8090/api/v3/hosts/snapshot/{host_id}/?start_time=1587449571.918&end_time=1587453171.918 花括号内的数据
    hostID := req.PathParameter("host_id")
}
```
## Go的错误处理
Go 语言通过内置的错误接口提供了非常简单的错误处理机制<br>

error类型是一个接口类型：
```go
type error interface {
    Error() string
}
```
我们可以在编码中通过实现 error 接口类型来生成错误信息<br>
函数通常在最后的返回值中返回错误信息。使用errors.New 可返回一个错误信息：
```go
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // 实现
}
```
在平时使用中，还可以调用fmt.Errorf()包来返回一个error类型的数据

```go
json.Unmarshal(body, &resdata)
if resdata["status"] != float64(0){
    err = fmt.Errorf("%s", resdata["msg"])
    return nil, err
}
```