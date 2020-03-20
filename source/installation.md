## 安装部署

​  以下给出的配置文件模版，为不同类型容器启动的配置模版，实际使用可以根据需求组合使用，也可以按照官方给出的组合建议依次配置启动，手册最后会给出配置示范用例 [⇧]()。

## 模块概览

- [nat](https://docs.chiron.one/installation.html#nat-)
- [chiron](https://docs.chiron.one/installation.html#chiron-)
- [browser](https://docs.chiron.one/installation.html#browser-)
- [browser_chiron](https://docs.chiron.one/installation.html#browser_chiron-)
- [browser_db](https://docs.chiron.one/installation.html#browser_db-)



## nat[⇧](https://docs.chiron.one/installation.html#模块概览)

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  nat:                                #服务名，同一yml文件中不可重复
    container_name: nat               #容器名，同一机器上容器名不可重复
    image: sjxqqq/chiron:1.0.2        #镜像
    tty: true                         #
    working_dir: /nat                 #工作目录
    environment:                      #环境变量
      NATPORT: "3200"                 #用于指定端口
    command: ["bash" , "start.sh"]    #容器启动默认执行的命令
    volumes:                          #挂载
      - ./chiron/:/nat                #把本地的存放nat启动脚本的路径挂载到容器上
    network_mode: "host"              #设置容器启动模式为host模式
```

#### 启动脚本：

```shell
#!/bin/bash
cp /program/nat ./
./nat $NATPORT
bash
```



## chiron[⇧](https://docs.chiron.one/installation.html#模块概览)

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  node:                                  
    container_name: node                 
    image: centos:7                      
    tty: true
    working_dir: /chiron/node1
    environment:
      CHR_CHAINID: "32768"               #指定链启动的chainid
      CHR_PASSWORD: "123456"             #账户密码
      CHR_RPC: "3"                       #开启的rpc等级
      CHR_HOST: "0.0.0.0"                #rpc监听的host
      CHR_PORT: "8101"                   #rpc监听port
      CHR_PPROF: "9001"                  #
      CHR_NAT: "127.0.0.1"               #连接nat的host
      CHR_NATPORT: "3200"                #连接nat的port
      CHR_APPLY: "heavy"                 #节点启动角色
      CHR_AHOST: "*"                     #允许访问rpc的host名单
      CHR_SK: ""                         #待创建账号的私钥
    command: ["bash" , "start.sh"]       
    volumes:
      - /root/medicine_chiron/:/chiron
    network_mode: "host"
```

#### 启动脚本：

```shell
#!/bin/bash
cp -r /program/contracts /program/chiron ./

pw=$CHR_PASSWORD
sk=$CHR_SK

if [ -z "sk" ]; then
         echo "newaccount -miner -password pw
 exit" | ./chiron console
else
         echo "importkey -miner -password pw -privatekey sk
 exit" | ./chiron console
fi
nohup ./gzv miner --chainid $CHR_CHAINID --password pw --rpc $CHR_RPC --host $CHR_HOST --port $CHR_PORT --pprof $CHR_PPROF --nat $CHR_NAT --natport $CHR_NATPORT --apply $CHR_APPLY --ahost $CHR_AHOST >>nohup.out &
bash
```



## browser（浏览器）[⇧](https://docs.chiron.one/installation.html#模块概览)

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  browser:
    container_name: browser
    image: centos:7
    tty: true
    working_dir: /browser
    environment:
      BR_RPCADDR: "127.0.0.1"                         #浏览器启动需要连接rpc的host
      BR_RPCPORT: "8104"                              #浏览器启动需要连接rpc的prot
      BR_DBADDR: "127.0.0.1"                          #数据库的host
      BR_DBUSER: "root"																#数据库的用户
      BR_DBPW: "123456"                               #数据库密码
      BR_DBNAME: "chiron"                             #数据库名
      BR_DBPORT: "3306"                               #数据库端口
    command: ["bash" , "start.sh"]
    volumes:
      - /root/medicine_chiron/browser/:/browser
    network_mode: "host"
    depends_on:
    	- browser_db
      - browser_chiron
```

#### 启动脚本：

```shell
#!/bin/bash
cp /program/browser ./
nohup ./browser --rpcaddr $BR_RPCADDR -rpcport $BR_RPCPORT -dbaddr $BR_DBADDR -dbuser $BR_DBUSER --dbpw $BR_DBPW --dbname $BR_DBNAME --dbport $BR_DBPORT >> nohup.out&
bash
```

提醒：浏览器节点和浏览器观察节点可放在同一个yml文件中使用，

## broswer_chiron（浏览器观察节点）[⇧](https://docs.chiron.one/installation.html#模块概览)

#### 配置文件模版：docker-compose.yml

```yml
version: '2'
services:
  browser_chiron:
    container_name: browser_chiron
    image: centos:7
    tty: true
    working_dir: /chiron
    environment:
      CHAINID: "32768"                         #观察节点连接的chainid
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
    command: ["bash" , "start.sh"]             
    volumes:
      - /root/medicine_chiron/browser_node/:/chiron
    network_mode: "host"
    depends_on:
      - browser_db
```

#### 启动脚本：

```shell
#!/bin/bash
cp -r /program/bro_chiron /program/contracts ./
chiron="chiron"
bro_Chiron="bro_chiron"
TARGET=""

pw=$PASSWORLD
sk=$ACCOUNTSK

if [ -z "sk" ]; then
         echo "newaccount -miner -password pw
 exit" | ./bro_Chiron console
else
         echo "importkey -miner -password pw -privatekey sk
 exit" | ./bro_Chiron console
fi
nohup ./bro_chiron miner --chainid $CHAINID --password pw --rpc $RPC --host $HOST --port $PORT --nat $NAT --natport $NATPORT --synctomysql --dbhost $DBHOST --dbname $DBNAME --dbuser $DBUSER --dbpassword $DBPASSWORLD >>nohup.out &

bash
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
mkdir /genesisFile
```

..............（本步骤生成的创世文件暂时由对接的开发提供）。

得到配置文件如下：chiron.cof  ,  genesis_msk.info ,contruct/

2、创建每种容器所需的配置和脚本文件夹，为挂载做准备（示范用例规模：创建一个nat节点容器，三个创世节点容器，一个mysql容器，一个browser容器，一个观察节点容器）

1）、创建个文件路径，执行以下命令：

```shell
mkdir /chiron
cd chiron/
mkdir nat
mkdir node1 
mkdir node2
mkdir node3 
mkdir mysql 
mkdir browser 
mkdir browser_node 
cp -r /genesisFile/keystore /genesisFile/chiron.cof /genesisFile/genesis_msk.info node1/
cp -r /genesisFile/keystore /genesisFile/chiron.cof /genesisFile/genesis_msk.info node2/
cp -r /genesisFile/keystore /genesisFile/chiron.cof /genesisFile/genesis_msk.info node3/
cp -r /genesisFile/keystore /genesisFile/chiron.cof /genesisFile/genesis_msk.info browser_node/
```

上述操作后，文件结构如图：

![image-20200317135431319](/Users/jiaxingsun/Library/Application Support/typora-user-images/image-20200317135431319.png)

2）、准备各节点启动脚本：（脚本名称：start.sh）

- 创建节点启动脚本：start.sh，把以下内容粘贴进去，然后把start.sh分别拷贝到node1，node2和node3文件夹中。

```shell
#!/bin/bash
cp -r /program/contracts /program/chiron ./

pw=$CHR_PASSWORD
nohup ./chiron miner --chainid $CHR_CHAINID --password pw --rpc $CHR_RPC --host $CHR_HOST --port $CHR_PORT --pprof $CHR_PPROF --nat $CHR_NAT --natport $CHR_NATPORT --apply $CHR_APPLY --ahost $CHR_AHOST -k $CHR_KEYSTORE >>nohup.out &
bash
```

- 创建nat启动脚本：start.sh,把以下内容粘贴进去，然后把start.sh拷贝到nat文件夹中。

```shell
#!/bin/bash
cp /program/nat ./
./nat $NATPORT
bash
```

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



- 创建browser启动脚本start.sh，把以下内容粘贴进start.sh，然后把start.sh拷贝到browser文件夹中。

```shell
#!/bin/bash
cp /program/browser ./
nohup ./browser --rpcaddr $BR_RPCADDR -rpcport $BR_RPCPORT -dbaddr $BR_DBADDR -dbuser $BR_DBUSER --dbpw $BR_DBPW --dbname $BR_DBNAME --dbport $BR_DBPORT >> nohup.out&
bash
```

- 创建观察节点启动脚本

```shell
#!/bin/bash
cp -r /program/bro_chiron /program/contracts ./
chiron="chiron"
bro_Chiron="bro_chiron"
TARGET=""

pw=$PASSWORLD
sk=$ACCOUNTSK

if [ -z "sk" ]; then
         echo "newaccount -miner -password pw
 exit" | ./bro_Chiron console
else
         echo "importkey -miner -password pw -privatekey sk
 exit" | ./bro_Chiron console
fi
nohup ./bro_chiron miner --chainid $CHAINID --password pw --rpc $RPC --host $HOST --port $PORT --nat $NAT --natport $NATPORT --synctomysql --dbhost $DBHOST --dbname $DBNAME --dbuser $DBUSER --dbpassword $DBPASSWORLD >>nohup.out &

bash
```



上述脚本准备完成之后目录结构应该如图所示：

![image-20200317141253747](/Users/jiaxingsun/Library/Application Support/typora-user-images/image-20200317141253747.png)

3、准备启动nat节点和创世节点的docker-compose.yml文件：

创建docker-compose.yml配置文件,把以下内容粘贴进去，然后把docker-compose.yml拷贝到/chiron文件夹中。

```yml
version: '2'
services:
  nat:                                #参数：服务名，同一yml文件中不可重复
    container_name: nat               #参数：容器名，同一机器上容器名不可重复
    image: sjxqqq/chiron:1.0.2        
    tty: true                         
    working_dir: /nat                 
    environment:                      #环境变量
      NATPORT: "3200"                 #参数：用于指定端口
    command: ["bash" , "start.sh"]    
    volumes:                          
      - /chiron/nat/:/nat             #参数：把本地的存放nat启动脚本的路径挂载到容器上
    network_mode: "host"              
    
  node1:                              #参数：服务名，同一yml文件中不可重复  
    container_name: node1             #参数：容器名，同一机器上容器名不可重复   
    image: centos:7                      
    tty: true
    working_dir: /node
    environment:
      CHR_CHAINID: "32768"               #参数：指定链启动的chainid
      CHR_PASSWORD: "123456"             #参数：账户密码
      CHR_RPC: "3"                       #参数：开启的rpc等级
      CHR_HOST: "0.0.0.0"                #参数：rpc监听的host
      CHR_PORT: "8101"                   #参数：rpc监听port
      CHR_PPROF: "9001"                  #参数：指定pprof分析工具开放端口
      CHR_NAT: "127.0.0.1"               #参数：连接nat的host
      CHR_NATPORT: "3200"                #参数：连接nat的port
      CHR_APPLY: "heavy"                 #参数：节点启动角色
      CHR_KEYSTORE: "keystore1"          #参数：指定keystore
    command: ["bash" , "start.sh"]       
    volumes:
      - /chiron/node1:/node              #参数：把本地的存放节点启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
      - nat
      
  node2:
    container_name: node2
    image: centos:7
    tty: true
    working_dir: /node
    environment:
      CHR_CHAINID: "32768"
      CHR_PASSWORD: "123456"
      CHR_RPC: "3"
      CHR_HOST: "0.0.0.0"
      CHR_PORT: "8102"
      CHR_PPROF: "9002"
      CHR_NAT: "127.0.0.1"
      CHR_NATPORT: "3200"
      CHR_APPLY: "heavy"
      CHR_KEYSTORE: "keystore2"
    command: ["bash" , "start.sh"]
    volumes:
      - /chiron/node2/:/node
    network_mode: "host"
    depends_on:
      - nat
      
  node3:
    container_name: node3
    image: centos:7
    tty: true
    working_dir: /node
    environment:
      CHR_CHAINID: "32768"
      CHR_PASSWORD: "123456"
      CHR_RPC: "3"
      CHR_HOST: "0.0.0.0"
      CHR_PORT: "8103"
      CHR_PPROF: "9003"
      CHR_NAT: "127.0.0.1"
      CHR_NATPORT: "3200"
      CHR_APPLY: "heavy"
      CHR_KEYSTORE: "keystore3"
    command: ["bash" , "start.sh"]
    volumes:
      - /chiron/node3:/node
    network_mode: "host"
    depends_on:
      - nat
      
  browser_db:
    container_name: browser_db
    image: mysql:5.7
    tty: true
    environment:
      MYSQL_ROOT_PASSWORD: "123456"         #参数：数据库密码
    volumes:                                #参数：以下两个挂载是把本地的存放mysql启动相关文件的路径挂载到容器上
      - /root/chiron/mysql/my.cnf:/etc/mysql/my.cnf
      - /root/chiron/mysql/chiron.sql:/docker-entrypoint-initdb.d/chiron.sql
    network_mode: "host"
  
  browser_node:
    container_name: browser_node
    image: centos:7
    tty: true
    working_dir: /chiron
    environment:
      CHAINID: "32768"                         #参数：观察节点连接的chainid
      PASSWORLD: "123123"                      #参数：账户密码
      RPC: "3"                                 #参数：rpc开放等级
      HOST: "0.0.0.0"                          #参数：rpc监听host
      PORT: "8104"                             #参数：rpc监听端口
      PPROF: "9004"                            #参数：指定pprof分析工具开放端口
      DBNAME: "chiron"                         #参数：数据库名
      DBUSER: "root"                           #参数：数据库用户名
      DBPASSWORLD: "123456"                    #参数：数据库密码
      DBHOST: "127.0.0.1"                      #参数：数据库host
      DBPORT: "3306"                           #参数：数据库端口
      NAT: "127.0.0.1"                         #参数：nat服务host
      NATPORT: "3200"                          #参数：nat服务端口
      ACCOUNTSK: ""                            #参数：待创建账号的私钥                     
    command: ["bash" , "start.sh"]             
    volumes:
      - /root/chiron/browser_node/:/chiron     #参数：把本地的存放观察节点启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
      - browser_db
      
  browser:
    container_name: browser
    image: centos:7
    tty: true
    working_dir: /browser
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
      - /root/chiron/browser/:/browser           #参数：把本地的存放浏览器启动脚本的路径挂载到容器上
    network_mode: "host"
    depends_on:
    	- browser_db
      - browser_node
```

4、创世节点启动相关yml文件和脚本已经启动完毕，在docker-compose.yml所在路径下执行以下命令：

```shell
docker-compose up -d --no-deps --build nat node1 node2 node3 
```

至此，创世节点已经启动完毕



5、准备浏览器相关容器启动

```shell
docker-compose up -d --no-deps --build browser_db browser_node browser 
```

至此，浏览器已经启动完毕





