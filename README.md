### Задание 1: подключить для тестового конфига общую папку
- Deploy [fronten и backend](./stage/Deployment.yml) в одном контейнере
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
frontwhithback-8b58fd66d-r2pdr   2/2     Running   0          54m
```
- Создание файла на backend
```
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl exec frontwhithback-8b58fd66d-r2pdr -c backend -- touch example/static/example.txt
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl exec frontwhithback-8b58fd66d-r2pdr -c backend -- ls -l /backend/static/
total 0
-rw-r--r-- 1 root root 0 Oct 20 17:00 example.txt
```

- Проверка наличия файла на frontend
```
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl exec frontwhithback-8b58fd66d-r2pdr -c frontend -- ls -l /frontend/static/
total 0
-rw-r--r-- 1 root root 0 Oct 20 17:00 example.txt
```

### Задание 2: подключить общую папку для прода

- Проверка наличия StorageClass после установки через helm
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl get storageclasses.storage.k8s.io 
NAME   PROVISIONER                                       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs    cluster.local/nfs-server-nfs-server-provisioner   Delete          Immediate           true                   52m
```

- Создание [PersistentVolumeClaim](./prod/PersistentVolumeClaim.yml) для подов 
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl get pvc
NAME   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc    Bound    pvc-17551857-c420-40a3-80eb-1a7333bb7291   2Gi        RWX            nfs            20m
```

- Deploy [frontend](./prod/DeploymentFrontend.yml) и [backend](./prod/DeploymentBackend.yml)
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
backend-59d7bd98f9-j6knb   1/1     Running   0          19m     10.233.105.131   worker1   <none>           <none>
backend-59d7bd98f9-pt9rw   1/1     Running   0          8m1s    10.233.105.132   worker1   <none>           <none>
backend-59d7bd98f9-svf7q   1/1     Running   0          8m1s    10.233.125.3     worker2   <none>           <none>
front-64c5fd45c7-fgd6t     1/1     Running   0          7m57s   10.233.125.4     worker2   <none>           <none>
front-64c5fd45c7-j4kwm     1/1     Running   0          7m57s   10.233.105.133   worker1   <none>           <none>
front-64c5fd45c7-kljxf     1/1     Running   0          19m     10.233.105.130   worker1   <none>           <none>
```

- Создание файла на одной node в pods backend-59d7bd98f9-svf7q и проверка его наличия на другой node через front-64c5fd45c7-j4kwm
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl exec backend-59d7bd98f9-svf7q -c backend -- touch to_front.txt /backend/static/to_front.txt
```
```
vlad@home:~/Documents/Netology/DevOps/dz_№61$ kubectl exec front-64c5fd45c7-kljxf -c frontend -- ls -l /frontend/static/
total 0
-rw-r--r-- 1 root root 0 Oct 20 17:32 to_front.txt
```