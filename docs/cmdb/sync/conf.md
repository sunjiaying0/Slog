# 解析 配置文件(.conf)参数

创建一个SynchronizeServer类型的数据<br>
其包含如下字段：
```
type SynchronizeServer struct {
	Core                    *backbone.Engine
	Config                  *options.Config
	Service                 *synchronizeService.Service
	synchronizeClientConfig chan synchronizeUtil.SychronizeConfig
}
```
其中 *options.Config  该类型是设置conf配置文件内容的结构体<br>
包含如下字段:
```
type Config struct {
	Names           []string              // 任务name
	exceptionDir    string                // 异常目录
	ConifgItemArray []*ConfigItem         // config item info 配置项信息 
	Trigger         TriggerTime           // TriggerTime 的结构为{triggertype role}
}
```

以上字段内容均可通过sync.conf得到

