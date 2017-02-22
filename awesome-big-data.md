## BI
redash.io
开源商业智能平台，展现的样子土。
安装在share2：使用了[provisioning script](https://raw.githubusercontent.com/getredash/redash/master/setup/ubuntu/bootstrap.sh)脚本安装，但不要直接git clone到/opt目录下，因为bootstrop脚本要自己创建/opt/redash/目录/。自动安装了nginx。

saiku
开源的OLAP浏览器

spagobi
安装在share2:/usr/webb/：从github库的release页面下载二进制包。解压后将bin和database目录下的*.sh脚本文件chmod为可执行。先安装了opeenjdk-8-jdk。


https://github.com/samizdatco/arbor
js库,图形可视化, 多节点的网状图,动画,自动调整形状

http://www.anychart.com/
扁平风格图标,挺漂亮; 同一网站还有anyMap,js地图

http://bokeh.pydata.org/en/latest/
功能强大的python图库,种类繁多、炫酷、现代

http://bokeh.pydata.org/en/latest/
气泡图，泡大小颜色不同，多个叠加

https://github.com/airbnb/superset
数据探索平台，直觉、可视、交互。python，规模大。 http://airbnb.io/superset/gallery.html

http://chartd.co/
视网膜级折线图转化为一个图片


# [数据工程](https://github.com/igorbarinov/awesome-data-engineering)
### Charts and Dashboards
http://www.zingchart.com/
太多的图表类型

http://smoothiecharts.org/
流式线图，监控用

https://github.com/plotly/dash
python实现的交互式应用,使用了socket.io

### Workflow
https://github.com/spotify/luigi
链接批量任务,带界面

### ELK Elastic Logstash Kibana
https://github.com/pblittle/docker-logstash
一个比较完善的logstash docker镜像,支持从网上某URL下载配置文件。

### docker
https://github.com/ClusterHQ/flocker
跨节点同步docker卷

https://github.com/weaveworks/weave
跨节点跨数据中心的虚拟网络

## 数据集
### 即时
https://github.com/Interana/eventsim
模拟web点击事件，依赖java8，scala实现，生成json格式的用户点击数据

[reddit](https://www.reddit.com/r/datasets/comments/3mk1vg/realtime_data_is_available_including_comments/)
一个云服务，生成服务器推送的SSE事件流（Content-Type:text/event-stream）

## 监控
https://github.com/prometheus/prometheus
监控系统和时间序列数据库

[数据工程生态系统](http://xyz.insightdataengineering.com/blog/pipeline_map.html)