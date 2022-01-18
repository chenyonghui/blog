### AAS V10 MB Monitoring Service.

MB Monitoring Service 为实现 “安全软件资源监控实现对下列涉密专用软件进行监控“的功能，应用服务器定期将自身的状态定期发送到一UDP服务器中，具体格式见需求文档。

使用方法为：

在`$DOMAIN_HOME/config/domain.xml`中

configs/config 元素下添加：
```xml
<mb-monitoring-service client-ip="", client-port="", server-ip="", server-port="", basic-cycle="", config-cycle="", status-cycle="", ></mb-monitoring-service>
```

属性介绍：

| 属性名字     | 属性介绍                                                     |
| ------------ | ------------------------------------------------------------ |
| client-ip    | 客户端IP（可不配置），默认取本机ip地址，如果取到127.0.0.1这样的值，则需要手动指定 |
| client-port  | 客户端端口，默认6888                                         |
| server-ip    | 接收数据的服务器IP，必须配置。                               |
| server-port  | 接收数据的服务器端口，必须配置。                             |
| basic-cycle  | 中间件基础信息采集周期。<br>配置格式：<br>数字（单位为秒）。<br>数字m （分钟）。<br>数字h （小时）。<br/>数字d （天）。<br/>数字w （周）。<br/>数字 M（月）。<br/>数字 l（月）。<br/>数字 y（年）。 |
| config-cycle | 中间件配置信息采集周期，格式同上。                           |
| status-cycle | 中间件状态信息采集周期，格式同上。                           |