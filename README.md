# kubernetes-open-api-doc

plugin to view on vs code is Markdown Preview Enhanced(cmd+k after key v)

## pods commands
```
kubectl apply -f pod-app-front.yaml 
kubectl get pods
kubectl describe pod openapi-front-pod
kubectl exec -it openapi-front-pod -- /bin/bash
kubectl exec -it openapi-front-pod -- ls /usr/share/nginx/html

```

## pods examples
for this the cluster have two nodes(node-a and node-b).
node-a have two pods:
    * pod-a have an app angualar
    * pod-b have an server nginx
node-b have the pod-c:
    * pod-c have on nginx app


### creating pod-a and pod-b into of the node-a and pod-c into of the node-b
creting label to node-a and node-b

```bash
kubectl get nodes -o wide

gke-clusterkubernetes-pool-1-19257034-5zl5   Ready    <none>   99m   v1.14.10-gke.27   10.128.0.8    34.70.33.23      Container-Optimized OS from Google   4.14.138+        docker://18.9.7
gke-clusterkubernetes-pool-1-19257034-nq3r   Ready    <none>   99m   v1.14.10-gke.27   10.128.0.7    35.238.156.105   Container-Optimized OS from Google   4.14.138+        docker://18.9.7

# we apply labels to nodes 
kubectl label nodes gke-clusterkubernetes-pool-1-19257034-5zl5 nodename=node-a
kubectl label nodes gke-clusterkubernetes-pool-1-19257034-nq3r nodename=node-b

# we validate labels of the nodes
kubectl get nodes --show-labels

# create pods
kubectl apply -f pods-example/pod-a.yaml
kubectl apply -f pods-example/pod-b.yaml
kubectl apply -f pods-example/pod-c.yaml

# we validate pods creation
kubectl get pods -o wide

pod-a   1/1     Running            0          61s   10.0.1.7   gke-clusterkubernetes-pool-1-19257034-5zl5   <none>           <none>
pod-b   0/1     ErrImagePull       0          55s   10.0.1.8   gke-clusterkubernetes-pool-1-19257034-5zl5   <none>           <none>
pod-c   0/1     ImagePullBackOff   0          50s   10.0.0.7   gke-clusterkubernetes-pool-1-19257034-nq3r   <none>           <none>
```

### we validate the network structure into nodes and pods
```
# we connect to pod-a
kubectl exec -it pod-a  -- sh

# install tool
apt update && apt upgrade && apt install -y curl net-tools traceroute iputils-ping

# get ip
hostname -I

10.0.1.7

# get gateway(10.0.1.1)
route -n 

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

### we connect to pod-b
```
# we connect to pod-a
kubectl exec -it pod-a  -- sh
# install tool
apt update && apt upgrade && apt install -y curl net-tools traceroute iputils-ping

# get ip
hostname -I

10.0.1.9

# get gateway(10.0.1.1)
route -n 

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

### we connect to pod-c(node-a)
```
# we connect to pod-c
kubectl exec -it pod-c  -- sh
# install tool
apt update && apt upgrade && apt install -y curl net-tools traceroute iputils-ping

# get ip
hostname -I

10.0.0.8

# get gateway(10.0.0.1)
route -n 

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.1        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

### resume ip address
it make sense with the result of the command
```
kubectl get pods -o wide
```
| node   | pod   | ip       | gateway  |   |
|--------|-------|----------|----------|---|
| node-a | pod-a | 10.0.1.7 | 10.0.1.1 |   |
| node-a | pod-b | 10.0.1.9 | 10.0.1.1 |   |
| node-b | pod-c | 10.0.0.8 | 10.0.0.1 |   |

## test connection between pods

```
# ping from a to b
ping 10.0.1.9

PING 10.0.1.9 (10.0.1.9) 56(84) bytes of data.
64 bytes from 10.0.1.9: icmp_seq=1 ttl=64 time=0.161 ms
64 bytes from 10.0.1.9: icmp_seq=2 ttl=64 time=0.060 ms

# ping from b to c
ping 10.0.0.8

PING 10.0.0.8 (10.0.0.8) 56(84) bytes of data.
64 bytes from 10.0.0.8: icmp_seq=1 ttl=62 time=1.39 ms
64 bytes from 10.0.0.8: icmp_seq=2 ttl=62 time=0.312 ms


# ping from b to a
ping 10.0.1.7

PING 10.0.1.7 (10.0.1.7) 56(84) bytes of data.
64 bytes from 10.0.1.7: icmp_seq=1 ttl=64 time=0.102 ms
64 bytes from 10.0.1.7: icmp_seq=2 ttl=64 time=0.069 ms

# ping from b to c
ping 10.0.0.8

PING 10.0.0.8 (10.0.0.8) 56(84) bytes of data.
64 bytes from 10.0.0.8: icmp_seq=1 ttl=62 time=1.60 ms
64 bytes from 10.0.0.8: icmp_seq=2 ttl=62 time=0.321 ms

# ping from c to a
ping 10.0.1.7

PING 10.0.1.7 (10.0.1.7) 56(84) bytes of data.
64 bytes from 10.0.1.7: icmp_seq=1 ttl=62 time=1.33 ms

# ping from c to b
# ping 10.0.1.9

PING 10.0.1.9 (10.0.1.9) 56(84) bytes of data.
64 bytes from 10.0.1.9: icmp_seq=1 ttl=62 time=1.39 ms
64 bytes from 10.0.1.9: icmp_seq=2 ttl=62 time=0.340 ms

# ping from c to ip unknow
ping 10.0.1.20

PING 10.0.1.20 (10.0.1.20) 56(84) bytes of data.
From 10.128.0.8 icmp_seq=1 Destination Host Unreachable
From 10.128.0.8 icmp_seq=2 Destination Host Unreachable

```

#### In resume we can say that all pods are correctly routings

### http protocol
if we make one http request from anyone pod to anyone pod the response it will success, the only condition is that port it is open in de container(EXPOSE <port_number>) and the container is listening the request(anyone server)

```
curl http://10.0.1.7

ok

31385

curl http://10.0.1.9:8080
curl: (7) Failed to connect to 10.0.1.9 port 8080: Connection refused
```
## TODO: hacer un traceroute para demostrar como se comunican los pods y como el cluster crea un router virtual para hacer posible la comunicacion entre diferentes lan

```bash
# we connect to pod c
traceroute ip-pod

```




# nodePort
we will crete 2 pods, in of first pod we going to deploy one application angular as long as that second pod we going to deploy an api rest (django)that contain it services for the angular app.

so that the app is exposed to internet we will have make a service of type nodePort for each app. the angular app wil have exposed the port 30080 while the app django will have exposed the port 30800.

So whit this 2 services we can to acceder both apps(angular app and django admin) but the angular app also it will connect to the api

All yaml files it is into service-nodeport-example folder 


```bash
# delete all pods
kubectl delete pods pod-a
kubectl delete pods pod-b
kubectl delete pods pod-c

# we create angular and django pods
kubectl apply -f service-nodeport-example/angular-pod.yaml
kubectl apply -f service-nodeport-example/django-pod.yaml 

# we validate that the pods they will are created successfully
kubectl get pods -o wide

AME          READY   STATUS    RESTARTS   AGE     IP          NODE                                         NOMINATED NODE   READINESS GATES
angular-pod   1/1     Running   0          2m20s   10.0.1.14   gke-clusterkubernetes-pool-1-19257034-5zl5   <none>           <none>
django-pod    1/1     Running   0          2m8s    10.0.0.15   gke-clusterkubernetes-pool-1-19257034-nq3r   <none>           <none>

# we will create both services
kubectl apply -f service-nodeport-example/angular-nodeport.yaml
kubectl apply -f service-nodeport-example/django-nodeport.yaml

# we will validate that the services they will make created successfully
kubectl get services -o wide

NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE    SELECTOR
angular-service   NodePort    10.64.4.128   <none>        80:30080/TCP     54s    app=test,type=frontend
django-service    NodePort    10.64.11.21   <none>        8000:30800/TCP   38s    app=openapi,type=backend
kubernetes        ClusterIP   10.64.0.1     <none>        443/TCP          3d3h   <none>

# we can to acceder to angular app throht of the ip of anyone node and the port 30080, for example http://34.70.33.23:30080
kubectl get nodes -o wide

NAME                                         STATUS   ROLES    AGE     VERSION           INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-clusterkubernetes-pool-1-19257034-5zl5   Ready    <none>   2d11h   v1.14.10-gke.27   10.128.0.8    34.70.33.23      Container-Optimized OS from Google   4.14.138+        docker://18.9.7
gke-clusterkubernetes-pool-1-19257034-nq3r   Ready    <none>   2d11h   v1.14.10-gke.27   10.128.0.7    35.238.156.105   Container-Optimized OS from Google   4.14.138+        docker://18.9.7

# also we can acceder to django admin with the ip of the anyone node and the port 30800, for example http://34.70.33.23:30800/admin/

```





# we make rule to firewall
gcloud compute firewall-rules create node-port-service --allow tcp:30080,tcp:30443

# get nodes/vm/instances to get node ip(get nodes also work)
gcloud compute instances list
NAME                                        ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
gke-clusterkubernetes-pool-1-19257034-5zl5  us-central1-c  e2-micro                   10.128.0.8   34.70.33.23     RUNNING
gke-clusterkubernetes-pool-1-19257034-nq3r  us-central1-c  e2-micro                   10.128.0.7   35.238.156.105  RUNNING

# the pod already is expose to external request(open in browser)
curl 34.70.33.23:30080
```

## pod-d backend api rest
```bash
kubectl apply -f pods-example/pod-d.yaml
kubectl get pods -o wide
kubectl exec -it pod-a  -- sh
curl http://10.0.0.10:8000

# desde fuera del cluster intentamos acceder al pod. no disponible
curl http://34.70.33.23:8000

# creamod un sericio nodeport para abrir ese puerto
kubectl apply -f pods-example/node-port-service-backend.yaml
# creamos regla en el firewall
gcloud compute firewall-rules create node-port-service-back --allow tcp:30800
# probamos disponibilidad desde caulquier nodo del cluster.
curl http://34.70.33.23:8000

# conectamos a pod para crear un usuario admin y poder acceder al admin de django

./manage.py createsuperuser
Username: ale@admin.com
Password: 
Password (again): 
```

TODO: modificar app angular para que apunte a la ip del pod backend


```bash
kubectl exec -it openapi-front-pod  -- sh
apt update && apt upgrade
apt install curl
apt install net-tools
 --- routing ip ---
netstat -nr
network services
netstat -pnltu
---- traceroute ---- on pod 2 same node . node 1 : pod-a -> 10.0.0.5, pod-b -> 10.0.0.6(same red lan) -------- node 2: pod c ->
apt install -y net-tools traceroute
traceroute 10.0.0.5
curl 10.0.0.5
```






gcloud container clusters describe clusterkubernetes