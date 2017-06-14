### 部署步骤
1. 上传代码至部署机器上;
              
2. 创建/usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering，并在该目录下解压代码包;
                             
3. 配置环境
              
 - 3.1 在部署机器上 vim /etc/rsyncd.conf，配置语句如下:                        
       [user_cluster]      
       path = /data1/meitu/meipai/user_cluster    
       ignore errors     
       read only = no     
       hosts allow = 192.168.0.0/16     
  并启动rsync服务;
                                    
 - 3.2 在部署机器上安装hive-client;
             
 - 3.3 在部署机器上启动crontab服务，"/sbin/service crond start";
 
 - 3.4 在部署机器上安装python 2.7, 首先用rsync拉取python2.7的安装文件，然后再进行安装:
 
       mkdir /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib/
 
       rsync -av root@192.168.137.180::lbw/python_install_files/Python-2.7.10.tar /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib/
 
       cd /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib
 
       tar -zxvf Python-2.7.10.tar
 
       cd Python-2.7.10
 
       ./configure
 
       make
 
       make install
 
 - 3.5 在部署机器上安装python的部分包：numpy (1.12.1)，scipy (0.19.0)，scikit-learn (0.18.1), 首先用rsync拉取所有python包的安装文件，然后再进行安装:
 
       rsync -av root@192.168.137.180::lbw/python_install_files/numpy-1.12.1.tar.gz /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib/
 
       rsync -av root@192.168.137.180::lbw/python_install_files/scipy-0.19.0.tar.gz /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib/
 
       rsync -av root@192.168.137.180::lbw/python_install_files/scikit-learn-0.18.1.tar.gz /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib/
 
       cd /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/lib
 
       tar -zxvf numpy-1.12.1.tar.gz
 
       tar -zxvf scipy-0.19.0.tar.gz
 
       tar -zxvf scikit-learn-0.18.1.tar.gz
 
       python2.7 numpy-1.12.1/setup.py install
 
       python2.7 scipy-0.19.0/setup.py install
 
       python2.7 scikit-learn-0.18.1/setup.py install
                      
4. 执行初始化, 初始化方法: 执行文件./sbin/install.sh;
                                                        

### 启动、关闭与卸载                     
#### 启动                  
 - 在部署机器(192.168.137.112)上，vim /etc/crontab里面添加如下配置：
         
       00 12 * * * root /bin/sh /usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/code/run_user_cluster.sh                      
 - 启动脚本位于/usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/sbin/start.sh，可手动进行启动;                        
 - 需要root权限;                    

#### 关闭              
  - 关闭程序/usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/sbin/stop.sh， 相当于kill；                       
  - 会保存前一天的结果数据作为备份，如果当天的数据需要回滚，需要启动/usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/sbin/rollback.sh，之后重新启动程序再跑今天的任务即可；             

#### 卸载
  - 代码包卸载脚本位于/usr/meitu/meipai/meipai_user_cluster/user_cluster/clustering/sbin/uninstall.sh，该脚本可删除所有相关程序文件和数据文件，日志文件会保留

### 检查点           
### 依赖组件与资源        
#### 部署资源            
 - 部署机器：192.168.137.112；              
 - 目标机器：16核超线程2.4GHz CPU；96G内存，1T硬盘；            
 - CPU需求：16核2.4G以上CPU；                
 - 内存需求：50G以内；             
 - 磁盘需求：正常情况数据占用近90G左右，至少预留120G磁盘空间；
                    
#### 依赖资源            
 - 初始化数据时，一次性的依赖192.168.137.180上的一些数据；             

### 监控                      
#### 业务日志                    
1. 跑取原始数据的日志        
    - 路径: /www/logs/meitu/meipai/meipai_user_cluster/user_cluster/clustering/downloaddata.log.YYYYMMDD，hive-client获取原始日志的log；                
    - 异常报警:  grep "ERROR"；程序中已经配置好会发送到邮箱lbw@meitu.com；                                         
    - 日志自动删除，最多保留60天；                    
2.  特征提取的日志                             
    - 路径：/www/logs/meitu/meipai/meipai_user_cluster/user_cluster/clustering/dataprocess.log.YYYYMMDD, 每天处理数据并提取特征的log；                       
    - 异常报警：grep "ERROR";程序中已经配置好会发送到邮箱lbw@meitu.com；                        
    - 日志自动进行删除，最多保留60天；                         

3.  预测数据所属类别的日志              
    - 路径：/www/logs/meitu/meipai/meipai_user_cluster/user_cluster/clustering/predict.log.YYYYMMDD,对数据进行预测并更新聚类中心和发现新类的log；                      
    - 异常报警：grep "ERROR"，程序中已经配置好会发送到邮箱lbw@meitu.com；                   
    - 日志自动进行删除，最多保留60天；                         
            
#### 数据文件
 - 数据根目录: /data1/meitu/meipai/meipai_user_cluster/user_cluster/clustering/
 - 结果文件：/data1/meitu/meipai/meipai_user_cluster/user_cluster/clustering/result/goodresults.YYYYMMDD和/data1/meitu/meipai/meipai_user_cluster/user_cluster/clustering/result/badresults.YYYYMMDD，保留60天                
 - 生成的中间数据文件程序会进行自动清理；            


