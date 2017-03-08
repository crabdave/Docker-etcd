
使用Docker镜像部署etcd集群
 
参考：
官方参考
https://github.com/coreos/etcd/blob/master/Documentation/op-guide/container.md#docker
etcd使用
http://blog.csdn.net/u010424605/article/details/44592533
docker网络配置
http://www.infoq.com/cn/articles/docker-network-and-pipework-open-source-explanation-practice/
 
etcd版本，当前最新版v3.1.0
Docker启动脚本
启动三个容器，指定每个容器的IP
本样例是将三个容器放在同一台物理机上进行实验，实际应用中需要部署到各自不同的物理机上（可使用--net=host标记让容器使用宿主机网络）。
使用docker inspect命令查看物理机上docker0的网关，然后配置三个ip地址HOST_1~3
使用三个容器别名etcd1~3
 
 
# For each machine
ETCD_VERSION=v3.1.0
TOKEN=my-etcd-token
CLUSTER_STATE=new
NAME_1=etcd-node-0
NAME_2=etcd-node-1
NAME_3=etcd-node-2
HOST_1=172.17.0.2
HOST_2=172.17.0.3
HOST_3=172.17.0.4
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380
 
# For node 1
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
docker run -d --name etcd1 quay.io/coreos/etcd:${ETCD_VERSION} \
    /usr/local/bin/etcd \
    --data-dir=data.etcd --name ${THIS_NAME} \
    --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
    --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
    --initial-cluster ${CLUSTER} \
    --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
 
# For node 2
THIS_NAME=${NAME_2}
THIS_IP=${HOST_2}
docker run -d --name etcd2 quay.io/coreos/etcd:${ETCD_VERSION} \
    /usr/local/bin/etcd \
    --data-dir=data.etcd --name ${THIS_NAME} \
    --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
    --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
    --initial-cluster ${CLUSTER} \
    --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
 
# For node 3
THIS_NAME=${NAME_3}
THIS_IP=${HOST_3}
docker run -d --name etcd3 quay.io/coreos/etcd:${ETCD_VERSION} \
    /usr/local/bin/etcd \
    --data-dir=data.etcd --name ${THIS_NAME} \
    --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
    --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
    --initial-cluster ${CLUSTER} \
    --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
    
 
通过HTTP接口访问etcd接口
 
查看版本
curl -L http://172.17.0.2:2379/version
*设置一个key的value
curl http://172.17.0.2:2379/v2/keys/message -XPUT -d value="Hello world"
*获取一个key的value
curl http://172.17.0.2:2379/v2/keys/message


