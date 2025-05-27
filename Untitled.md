$0   文本当前行的全部内容
$1   文本的第1列
$2   文件的第2列
$3   文件的第3列，依此类推
NR   文件当前行的行号
NF   文件当前行

```bash
$0   文本当前行的全部内容
$1   文本的第1列
$2   文件的第2列
$3   文件的第3列，依此类推
NR   文件当前行的行号
NF   文件当前行的列数（有几列）
FS   字段分隔符，默认是空格
FNR  各文件分别记行号
FILENAME  文件名
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
BEGIN{ }    行前处理，读取文件内容前执行，指令执行1次
  { }         逐行处理，读取文件过程中执行，指令执行n次
  END{ }     行后处理，读取文件结束后执行，指令执行1次
  
```

awk 'BEGIN{ commands } pattern{ commands } END{ commands }'

```bash
#确认GitLab主机硬件配置
[root@GitLab ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3896         113        3691           8          90        3615
Swap:             0           0           0

#安装解压软件
[root@GitLab ~]# yum -y install unzip

#解压实验素材（提前从server1主机/linux-soft/s3/PROJECT02.zip拷贝的资料）
[root@GitLab ~]# unzip PROJECT02.zip

#查看解压的素材资料
[root@GitLab ~]# ls PROJECT02/GitLab/
gitlab-ce-12.4.6-ce.0.el7.x86_64.rpm

#安装GitLab软件包,强制忽略依赖安装
[root@GitLab ~]# rpm -ivh --nodeps gitlab-ce-12.4.6-ce.0.el7.x86_64.rpm                 
#重载GitLab配置（需要耐心等待）
[root@GitLab ~]# gitlab-ctl reconfigure
...
Running handlers:
Running handlers complete
Chef Client finished, 527/1423 resources updated in 02 minutes 05 seconds
gitlab Reconfigured!

#重启GitLab相关服务
[root@GitLab ~]# gitlab-ctl restart
ok: run: alertmanager: (pid 1975) 0s        //报警服务
ok: run: gitaly: (pid 1986) 1s              //Git后台服务
ok: run: gitlab-exporter: (pid 2014) 0s     //Prometheus数据采集器
ok: run: gitlab-workhorse: (pid 2020) 1s    //反向代理服务器
ok: run: grafana: (pid 2117) 0s             //数据可视化服务
ok: run: logrotate: (pid 2129) 0s           //日志文件管理服务
ok: run: nginx: (pid 2135) 1s               //静态WEB服务
ok: run: node-exporter: (pid 2142) 0s       //Prometheus数据采集器
ok: run: postgres-exporter: (pid 2148) 1s   //Prometheus数据采集器
ok: run: postgresql: (pid 2159) 0s          //数据库服务
ok: run: prometheus: (pid 2168) 0s          //Prometheus监控服务
ok: run: redis: (pid 2178) 1s               //缓存数据库服务
ok: run: redis-exporter: (pid 2183) 0s      //Prometheus数据采集器
ok: run: sidekiq: (pid 2192) 0s             //异步执行队列服务
ok: run: unicorn: (pid 2203) 0s             //Rails托管WEB服务
[root@GitLab ~]# 
```

