# 安装部署

​		以下给出的配置文件模版，为不同类型容器启动的配置模版，实际使用可以根据需求组合使用，也可以按照官方给出的组合建议依次配置启动，手册最后会给出配置示范用例 [⇧]()。

## 模块概览

- [nat](https://docs.chiron.one/installation.html#nat-)
- [chiron](https://docs.chiron.one/installation.html#chiron-)
- [browser](https://docs.chiron.one/installation.html#browser-)
- [browser_node](https://docs.chiron.one/installation.html#browser_chiron-)
- [browser_db](https://docs.chiron.one/installation.html#browser_db-)



## nat

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  nat:                                #服务名，同一yml文件中不可重复
    container_name: nat               #容器名，同一机器上容器名不可重复
    image: sjxqqq/chiron:1.0.1        #镜像
    tty: true                         #
    working_dir: /program             #工作目录
    environment:                      #环境变量
      NATPORT: "3200"                 #用于指定端口
    command: ["bash" , "start_nat.sh"] #容器启动默认执行的命令
    network_mode: "host"              #设置容器启动模式为host模式
```

#### 启动脚本：

```shell
#!/bin/bash
./nat $NATPORT
bash
```



## chiron

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  node:                                  
    container_name: node                 
    image: centos:7                      
    tty: true
    working_dir: /chiron/
    environment:
      CHR_CHAINID: "123"                 #指定链启动的chainid
      CHR_PASSWORD: "123456"             #账户密码
      CHR_RPC: "3"                       #开启的rpc等级
      CHR_HOST: "0.0.0.0"                #rpc监听的host
      CHR_PORT: "8101"                   #rpc监听port
      CHR_NAT: "127.0.0.1"               #连接nat的host
      CHR_NATPORT: "3200"                #连接nat的port
      CHR_APPLY: "heavy"                 #节点启动角色
      #CHR_AHOST: "*"                     #允许访问rpc的host名单,（此功能暂时关闭）
      ACCOUNTSK: ""                      #待创建账号的私钥
    command: /bin/bash -c 'cp /program/start_node.sh ./;./start_node.sh 1;'       
    volumes:
      - /root/medicine_chiron/node1/:/chiron
    network_mode: "host"
```

#### 启动脚本start_node.sh：

（目前节点分为普通节点和观察节点，此脚本可以启动这两类及节点，参数1表示普通节点，2表示表示观察节点,举例：准备启动一个普通节点，输入 `start_node.sh 1` 即可启动）

```shell
#!/bin/bash
TARGET=""
pw=$PASSWORLD
sk=$ACCOUNTSK
AUTOPORT=""

function Listening {
for k in $( seq 1 100 )
do
    AUTOPORT=900$k
    tomcat=`netstat -an | grep ":$AUTOPORT" | awk '$1 == "tcp6" && $NF == "LISTEN" {print $0}' | wc -l`
    if [ $tomcat -eq 0 ];then
        echo "拿到可用端口：$AUTOPORT"
        break
    else
        echo $AUTOPORT
    fi
done
}

#判断是普通节点还是观察节点
if [ "$1" = "1" ]; then
 pw=$CHR_PASSWORD
 cp -r /program/gch /program/contracts ./
 TARGET="gch"
elif [ "$1" = "2" ]; then
 cp -r /program/bro_gch /program/contracts ./
 TARGET="bro_gch"
fi

if [ -z "$ACCOUNTSK" ];then
        echo "newaccount -miner -password $pw
exit" | ./$TARGET console
else
        echo "importkey -miner -password $pw -privatekey $sk
exit" | ./$TARGET console
fi

Listening
#判断是普通节点还是观察节点
if [ "$1" = "1" ]; then
 pw=$CHR_PASSWORD
 nohup ./$TARGET miner --chainid $CHR_CHAINID --password $pw --rpc $CHR_RPC --host $CHR_HOST --port $CHR_PORT --pprof $AUTOPORT --nat $CHR_NAT --natport $CHR_NATPORT --apply $CHR_APPLY  >>nohup.out &
elif [ "$1" = "2" ]; then
 nohup ./$TARGET miner --chainid $CHAINID --password $pw --rpc $RPC --host $HOST --port $PORT --nat $NAT --natport $NATPORT --synctomysql --dbhost $DBHOST --dbport $DBPORT --dbname $DBNAME --dbuser $DBUSER --dbpassword $DBPASSWORLD >>nohup.out &
fi
bash
```



## browser（浏览器）

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  browser:
    container_name: browser
    image: centos:7
    tty: true
    working_dir: /chiron
    environment:
      BR_RPCADDR: "127.0.0.1"                         #浏览器启动需要连接rpc的host
      BR_RPCPORT: "8104"                              #浏览器启动需要连接rpc的prot
      BR_DBADDR: "127.0.0.1"                          #数据库的host
      BR_DBUSER: "root"																#数据库的用户
      BR_DBPW: "123456"                               #数据库密码
      BR_DBNAME: "chiron"                             #数据库名
      BR_DBPORT: "3306"                               #数据库端口
    command: ["bash" , "start_browser.sh"]
    volumes:
      - /root/medicine_chiron/browser/:/chiron
    network_mode: "host"
    depends_on:
    	- browser_db
      - browser_chiron
```

#### 启动脚本：

```shell
#!/bin/bash
cp -r /program/browser /program/webser ./
nohup ./browser --rpcaddr $BR_RPCADDR -rpcport $BR_RPCPORT -dbaddr $BR_DBADDR -dbuser $BR_DBUSER --dbpw $BR_DBPW --dbname $BR_DBNAME --dbport $BR_DBPORT >> nohup.out&
nohup ./webser >> web.out&
bash
```

提醒：浏览器节点和浏览器观察节点可放在同一个yml文件中使用，

## broswer_chiron（浏览器观察节点）

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  browser_node:
    container_name: browser_node
    image: centos:7
    tty: true
    working_dir: /chiron
    environment:
      CHAINID: "123"                           #观察节点连接的chainid
      PASSWORLD: "123123"                      #账户密码
      RPC: "3"                                 #rpc开放等级
      HOST: "0.0.0.0"                          #rpc监听host
      PORT: "8104"                             #rpc监听端口
      PPROF: "9004"                            #
      DBNAME: "chiron"                         #数据库名
      DBUSER: "root"                           #数据库用户名
      DBPASSWORLD: "123456"                    #数据库密码
      DBHOST: "127.0.0.1"                      #数据库host
      DBPORT: "3306"                           #数据库端口
      NAT: "127.0.0.1"                         #nat服务host
      NATPORT: "3200"                          #nat服务端口
      ACCOUNTSK: ""                            #待创建账号的私钥                     
    command: /bin/bash -c 'cp /program/start_node.sh ./;./start_node.sh 1;'             
    volumes:
      - /root/chiron/browser_node/:/chiron
    network_mode: "host"
    depends_on:
      - browser_db
```

#### 启动脚本：同上chiron节点启动脚本相同(节点启动脚本使用同一脚本)

```
略
```



## mysql[⇧](https://docs.chiron.one/installation.html#模块概览)

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  browser_db:
    container_name: browser_db
    image: mysql:5.7
    tty: true
    environment:
      MYSQL_ROOT_PASSWORD: "123456"         #数据库密码
    volumes:
      - /root/chiron/mysql/my.cnf:/etc/mysql/my.cnf
      - /root/chiron/mysql/chiron.sql:/docker-entrypoint-initdb.d/chiron.sql
    network_mode: "host"
```

### 需要挂载的配置文件：

#### chiron.sql

```sql
drop database if exists `chiron`;
create database `chiron` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

#### my.cnf

```cnf
[mysqld]
character_set_server=utf8mb4
init_connect='SET NAMES utf8'
wait_timeout=1814400
interactive_timeout=604800

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
[client]
default-character-set=utf8mb4
```



## 示范用例：

1、准备创建：创世配置文件

创建创世节点配置文件目录：

```shell
mkdir -p /chiron/genesisFile
cd /chiron/genesisFile
wget http://39.101.192.249:7788/chiron.tar
docker load < /chiron/genesisFile/chiron.tar
docker tag b8f0a8db8578 sjxqqq/chiron:1.0.1
wget http://39.101.192.249:7788/genesis_conf.zip
unzip genesis_conf.zip
```

得到配置文件如下：chiron.cof  ,  genesis_msk.info 

2、创建每种容器所需的配置和脚本文件夹，为挂载做准备（示范用例规模：创建一个nat节点容器，三个创世节点容器，一个mysql容器，一个browser容器，一个观察节点容器）

1）、创建个文件路径，执行以下命令：

```shell
cd /chiron/
mkdir node1 node2 node3 mysql browser browser_node  
cp -r /genesisFile/chiron.cof /genesisFile/genesis_msk.info node1/
cp -r /genesisFile/chiron.cof /genesisFile/genesis_msk.info node2/
cp -r /genesisFile/chiron.cof /genesisFile/genesis_msk.info node3/
cp -r /genesisFile/chiron.cof /genesisFile/genesis_msk.info browser_node/
```

上述操作后，文件结构如图：

![](install1.png)

2）、准备各mysql启动相关配置文件：

- 创建mysql节点启动相关配置，然后把以下文件拷贝到mysql文件夹中。

#### chiron.sql

```sql
drop database if exists `chiron`;
create database `chiron` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

#### my.cnf

```cnf
[mysqld]
character_set_server=utf8mb4
init_connect='SET NAMES utf8'
wait_timeout=1814400
interactive_timeout=604800

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
[client]
default-character-set=utf8mb4
```

以上操作准备完成后，结构最终如下：

![](install2.png)



3、准备启动nat节点和创世节点的docker-compose01.yml文件：(创世节点是普通节点的一种，是chiron链初始启动的三个节点)

1)创建docker-compose01.yml配置文件,把以下内容粘贴进去，然后把docker-compose01.yml拷贝到/chiron文件夹中。

```yml
version: '2'
services:
  nat:                                #参数：服务名，同一yml文件中不可重复
    container_name: nat               #参数：容器名，同一机器上容器名不可重复
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /program
    environment:                      #环境变量
      NATPORT: "3200"                 #参数：用于指定端口
    command: ["bash" , "start_nat.sh"]
    network_mode: "host"

  node1:                              #参数：服务名，同一yml文件中不可重复
    container_name: node1             #参数：容器名，同一机器上容器名不可重复
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /chiron
    environment:
      CHR_CHAINID: "123"                 #参数：指定链启动的chainid
      CHR_PASSWORD: "123456"             #参数：账户密码
      CHR_RPC: "3"                       #参数：开启的rpc等级
      CHR_HOST: "0.0.0.0"                #参数：rpc监听的host
      CHR_PORT: "8101"                   #参数：rpc监听port
      CHR_NAT: "127.0.0.1"               #参数：连接nat的host
      CHR_NATPORT: "3200"                #参数：连接nat的port
      CHR_APPLY: "heavy"                 #参数：节点启动角色
      ACCOUNTSK: "0x7d45c0be7872adda7b4abbdbd09ed5cf3cbc469f4b2e1a9f5724073638383450"
    command: >
      /bin/bash -c '
      cp /program/start_node.sh ./;
      ./start_node.sh 1;
      '
    volumes:
      - /chiron/node1:/chiron              #参数：把本地的存放节点启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
      - nat
  node2:
    container_name: node2
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /chiron
    environment:
      CHR_CHAINID: "32768"
      CHR_PASSWORD: "123456"
      CHR_RPC: "3"
      CHR_HOST: "0.0.0.0"
      CHR_PORT: "8102"
      CHR_NAT: "127.0.0.1"
      CHR_NATPORT: "3200"
      CHR_APPLY: "heavy"
      ACCOUNTSK: "0x7a6a57a885e60cf62fc3ec49666819a0e4c1f419074db3493a65831863bb529f"
    command: >
      /bin/bash -c '
      cp /program/start_node.sh ./;
      ./start_node.sh 1;
      '
    volumes:
      - /chiron/node2/:/chiron
    network_mode: "host"
    depends_on:
      - nat
      - node1

  node3:
    container_name: node3
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /chiron
    environment:
      CHR_CHAINID: "32768"
      CHR_PASSWORD: "123456"
      CHR_RPC: "3"
      CHR_HOST: "0.0.0.0"
      CHR_PORT: "8103"
      CHR_NAT: "127.0.0.1"
      CHR_NATPORT: "3200"
      CHR_APPLY: "heavy"
      ACCOUNTSK: "0x8d090a81d29ead88d739d18e7cc456f9e23061dd68bcc4eca55d6b4b136dcd39"
    command: >
      /bin/bash -c '
      cp /program/start_node.sh ./;
      ./start_node.sh 1;
      '
    volumes:
      - /chiron/node3:/chiron
    network_mode: "host"
    depends_on:
      - nat
      - node1 
      - node2
```

2)创世节点启动相关yml文件和脚本已经启动完毕，在docker-compose.yml所在路径下执行以下命令：

```shell
docker-compose up -d -f docker-compose01.yml
```

至此，创世节点已经启动完毕

4、准备浏览器启动相关docker-compose02.yml文件

```yml
version: '2'
services:  
  browser_db:
    container_name: browser_db
    image: mysql:5.7
    tty: true
    environment:
      MYSQL_ROOT_PASSWORD: "123456"         #参数：数据库密码
    volumes:                                #参数：以下两个挂载是把本地的存放mysql启动相关文件的路径挂载到容器上
      - /root/chiron/mysql/my.cnf:/etc/mysql/my.cnf
      - /root/chiron/mysql/chiron.sql:/docker-entrypoint-initdb.d/chiron.sql
      - /root/chiron/mysql:/var/lib/mysql-files/
    network_mode: "host"
  
  browser_node:
    container_name: browser_node
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /chiron
    environment:
      CHAINID: "32768"                         #参数：观察节点连接的chainid
      PASSWORLD: "123123"                      #参数：账户密码
      RPC: "3"                                 #参数：rpc开放等级
      HOST: "0.0.0.0"                          #参数：rpc监听host
      PORT: "8104"                             #参数：rpc监听端口
      DBNAME: "chiron"                         #参数：数据库名
      DBUSER: "root"                           #参数：数据库用户名
      DBPASSWORLD: "123456"                    #参数：数据库密码
      DBHOST: "127.0.0.1"                      #参数：数据库host
      DBPORT: "3306"                           #参数：数据库端口
      NAT: "127.0.0.1"                         #参数：nat服务host
      NATPORT: "3200"                          #参数：nat服务端口
      ACCOUNTSK: ""                            #参数：待创建账号的私钥                     
    command: >
      /bin/bash -c '
      cp /program/start_node.sh ./;
      ./start_node.sh 2;
      '             
    volumes:
      - /chiron/browser_node/:/chiron     #参数：把本地的存放观察节点启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
      - browser_db
      
  browser:
    container_name: browser
    image: sjxqqq/chiron:1.0.1
    tty: true
    working_dir: /chiron
    environment:
      BR_RPCADDR: "127.0.0.1"                     #参数：浏览器启动需要连接rpc的host
      BR_RPCPORT: "8104"                          #参数：浏览器启动需要连接rpc的prot
      BR_DBADDR: "127.0.0.1"                      #参数：数据库的host
      BR_DBUSER: "root"									   				#参数：数据库的用户
      BR_DBPW: "123456"                           #参数：数据库密码
      BR_DBNAME: "chiron"                         #参数：数据库名
      BR_DBPORT: "3306"                           #参数：数据库端口
    command: ["bash" , "start.sh"]
    volumes:
      - /chiron/browser/:/chiron           #参数：把本地的存放浏览器启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
    	- browser_db
      - browser_node
```

浏览器节点启动相关yml文件已经准备完毕，在docker-compose02.yml所在路径下执行以下命令：

```shell
docker-compose up -d -f docker-compose02.yml
```



