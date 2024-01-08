# Домашнее задание к занятию «Запуск приложений в K8S»

<br>

## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создание deployment с двумя репликами:

```
kubectl describe deployments.apps nginx-multitool 
Name:                   nginx-multitool
Namespace:              default
CreationTimestamp:      Mon, 08 Jan 2024 12:59:24 +0900
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-multitool
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-multitool
  Containers:
   nginx:
    Image:        nginx
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
   multitool:
    Image:       wbitt/network-multitool
    Ports:       1180/TCP, 11443/TCP
    Host Ports:  0/TCP, 0/TCP
    Requests:
      cpu:     1m
      memory:  20Mi
    Environment:
      HTTP_PORT:   1180
      HTTPS_PORT:  11443
    Mounts:        <none>
  Volumes:         <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-multitool-7945fdd44 (2/2 replicas created)
Events:          <none>
```
2. Проверка работы service из отдельного pod

```
kubectl exec --stdin --tty multitool -- /bin/bash
multitool:/# curl http://10.1.13.162:1180
WBITT Network MultiTool (with NGINX) - nginx-multitool-7945fdd44-zrxwl - 10.1.13.162 - HTTP: 1180 , HTTPS: 11443 . (Formerly praqma/network-multitool)
```

## Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создание deployment с init контейнером. Проверка статуса запущенного deployment'а 

```
kubectl get pod
NAME              READY   STATUS     RESTARTS   AGE
nginx-with-init   0/1     Init:0/1   0          21s

kubectl describe -f deployment2.yaml 
Name:             nginx-with-init
Namespace:        default
Priority:         0
Service Account:  default
Node:             homework/192.168.1.102
Start Time:       Mon, 08 Jan 2024 16:03:27 +0900
Labels:           app=nginx
Annotations:      cni.projectcalico.org/containerID: 1db7c1841dbffec2c3afc2deba93b9c36dec31d9668e03980b76555c28ddb691
                  cni.projectcalico.org/podIP: 10.1.13.167/32
                  cni.projectcalico.org/podIPs: 10.1.13.167/32
Status:           Pending
IP:               10.1.13.167
IPs:
  IP:  10.1.13.167
Init Containers:
  multitool:
    Container ID:  containerd://2ade4fdc29efbb9cc5bc1dffde1ac96e450e37c2232beb9176b4f884ca4b45d2
    Image:         wbitt/network-multitool
    Image ID:      docker.io/wbitt/network-multitool@sha256:d1137e87af76ee15cd0b3d4c7e2fcd111ffbd510ccd0af076fc98dddfc50a735
    Ports:         1180/TCP, 11443/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      sh
      -c
      until nslookup nginx.default.svc.cluster.local; do echo waiting for mydb; sleep 2; done
    State:          Running
      Started:      Mon, 08 Jan 2024 16:03:30 +0900
    Ready:          False
    Restart Count:  0
    Requests:
      cpu:     1m
      memory:  20Mi
    Environment:
      HTTP_PORT:   1180
      HTTPS_PORT:  11443
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l8jr9 (ro)
Containers:
  nginx:
    Container ID:   
    Image:          nginx
    Image ID:       
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l8jr9 (ro)
Conditions:
  Type              Status
  Initialized       False 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-l8jr9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  36s   default-scheduler  Successfully assigned default/nginx-with-init to homework
  Normal  Pulling    36s   kubelet            Pulling image "wbitt/network-multitool"
  Normal  Pulled     34s   kubelet            Successfully pulled image "wbitt/network-multitool" in 2.355s (2.355s including waiting)
  Normal  Created    34s   kubelet            Created container multitool
  Normal  Started    34s   kubelet            Started container multitool
```

2. Запуск сервиса. Проверка работы deployment

```
kubectl apply -f service2.yaml 
service/nginx created

kubectl get pod
NAME              READY   STATUS    RESTARTS   AGE
nginx-with-init   1/1     Running   0          7m41s

kubectl describe -f deployment2.yaml 
Name:             nginx-with-init
Namespace:        default
Priority:         0
Service Account:  default
Node:             homework/192.168.1.102
Start Time:       Mon, 08 Jan 2024 16:03:27 +0900
Labels:           app=nginx
Annotations:      cni.projectcalico.org/containerID: 1db7c1841dbffec2c3afc2deba93b9c36dec31d9668e03980b76555c28ddb691
                  cni.projectcalico.org/podIP: 10.1.13.167/32
                  cni.projectcalico.org/podIPs: 10.1.13.167/32
Status:           Running
IP:               10.1.13.167
IPs:
  IP:  10.1.13.167
Init Containers:
  multitool:
    Container ID:  containerd://2ade4fdc29efbb9cc5bc1dffde1ac96e450e37c2232beb9176b4f884ca4b45d2
    Image:         wbitt/network-multitool
    Image ID:      docker.io/wbitt/network-multitool@sha256:d1137e87af76ee15cd0b3d4c7e2fcd111ffbd510ccd0af076fc98dddfc50a735
    Ports:         1180/TCP, 11443/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      sh
      -c
      until nslookup nginx.default.svc.cluster.local; do echo waiting for mydb; sleep 2; done
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 08 Jan 2024 16:03:30 +0900
      Finished:     Mon, 08 Jan 2024 16:10:17 +0900
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:     1m
      memory:  20Mi
    Environment:
      HTTP_PORT:   1180
      HTTPS_PORT:  11443
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l8jr9 (ro)
Containers:
  nginx:
    Container ID:   containerd://48f9aa6915fe1700aabdd3e7f350f8c632df4079d8146dc59a3b380b99af36c1
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:2bdc49f2f8ae8d8dc50ed00f2ee56d00385c6f8bc8a8b320d0a294d9e3b49026
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 08 Jan 2024 16:10:20 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l8jr9 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-l8jr9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  59s   kubelet  Pulling image "nginx"
  Normal  Pulled   56s   kubelet  Successfully pulled image "nginx" in 2.367s (2.367s including waiting)
  Normal  Created  56s   kubelet  Created container nginx
  Normal  Started  56s   kubelet  Started container nginx
```