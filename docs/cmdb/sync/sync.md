# sync进程

* 所属项目：cmdb

### cmdb(蓝鲸配置平台)是一个面向资产及应用的企业级配置管理平台
    

* 其提供了全新的自定义模型管理，还可以根据不同的企业需求随时新增模型和关联关系，把网络/中间件/虚拟资源等纳入到CMDB的管理中，除此之外还增加了更多符合场景需要的新功能：机器数据快照/数据自动发现/变更事件主动推送/更加精细的权限管理/可拓展的的业务拓扑等功能


### 原sync进程同步功能
* 对于分支机构较多的公司，由于各种条件的限制，可能会有多套cmdb平台，每个分支都有自己的资产信息，要把分支CMDB的资产信息同步到总部，就需要配置CMDB的同步功能
* 如果环境中有多台CMDB角色的机器，则需要在每台CMDB上进行配置



### 现下cmdb项目中 sync进程的任务<br>
* 根据需求更改配置文件 sync.conf
    🔗 [sync配置](cmdb/sync/sync_config.md)<br>
* 定时调用topo的同步接口<br>


### sync进程代码解析：<br>

* sync进程的main入口： GOPATH/configcenter/src/scene_server/synchronize_server/synchronize.go<br>
* 再一步进入 run()函数：<br>
    run()函数中会解析参数 判断必要参数等<br>
    参数没有问题会进入 go synchronSrv.Service.InitBackground() 初始化背景任务<br>
    注册路由(该项目中使用的是go-restful框架实现的)<br>
    
* 再往程序里进一步：synchronSrv.Service.InitBackground()<br>
    InitBackground()函数会调用 同步触发函数TriggerSynchronize(）<br>
    该函数会进行配置文件解析，判断参数是否有效 🔗 [sync配置文件解析](cmdb/sync/conf.md)<br>
        * 先判断config是否为空
        * 再判断config中任务名是否为空（现在的项目不需要该判断 所以暂时注释
        * 判断定时任务设定时间是否为空（为空则为默认值），以及定时的类型
        * 调用定时任务处理函数 	syncfunc("test") 这里是暂时直接写死的参数，后面需要将参数model=“xxx”放在配置文件里处理
* 定时任务函数 syncfunc(model)：
    是一个调用topo同步任务接口的函数,极其简单<br>
    ```go
    func syncfunc(model string) error {
	//blog.InfoJSON("start topo_sync task : 【%s】", model)
	// jiajia todo 调用topo同步接口
	var err error

	info := make(map[string]string)
	info["model"] = model
	bytesData, err := json.Marshal(info)
	if err != nil {
		fmt.Println(err.Error())
		return err
	}
	reader := bytes.NewReader(bytesData)

	url := "http://127.0.0.1:60002/topo/v3/internal/model_task"
	request, err := http.NewRequest("POST", url, reader)
	if err != nil {
		// handle error
		blog.Errorf("Synchronize topo error:%s",err)
	}

	request.Header.Set("Content-Type", "application/json;charset=UTF-8")
	defer request.Body.Close() //程序在使用完回复后必须关闭回复的主体

	client := http.Client{}
	resp, err := client.Do(request) //Do 方法发送请求，返回 HTTP 回复

	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		// handle error
		blog.Errorf("handle Synchronize_topo error:%s",err)
	}
	return nil
    }
    ```
	当然 ⬆️的方法是入门级的我想出来的，实际上这样的方法并不优美<br>
	蓝鲸cmdb的项目里 自己有写一套调取内部api的方法的<br>
    

简单的get请求
func main(){
resp, err := http.Get("http://192.168.23.16:8000/api/v1/salt/minion-status")
    if err != nil {
        panic(err)
    
    }
    defer resp.Body.Close()
    s,err:=ioutil.ReadAll(resp.Body)
    fmt.Printf(string(s))
}