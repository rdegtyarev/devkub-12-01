# Домашнее задание к занятию "12.1 Компоненты Kubernetes"

Вы DevOps инженер в крупной компании с большим парком сервисов. Ваша задача — разворачивать эти продукты в корпоративном кластере. 

## Задача 1: Установить Minikube
<details>

  <summary>Описание задачи</summary>  
Для экспериментов и валидации ваших решений вам нужно подготовить тестовую среду для работы с Kubernetes. Оптимальное решение — развернуть на рабочей машине Minikube.

### Как поставить на AWS:
- создать EC2 виртуальную машину (Ubuntu Server 20.04 LTS (HVM), SSD Volume Type) с типом **t3.small**. Для работы потребуется настроить Security Group для доступа по ssh. Не забудьте указать keypair, он потребуется для подключения.
- подключитесь к серверу по ssh (ssh ubuntu@<ipv4_public_ip> -i <keypair>.pem)
- установите миникуб и докер следующими командами:
  - curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
  - chmod +x ./kubectl
  - sudo mv ./kubectl /usr/local/bin/kubectl
  - sudo apt-get update && sudo apt-get install docker.io conntrack -y
  - curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
- проверить версию можно командой minikube version
- переключаемся на root и запускаем миникуб: minikube start --vm-driver=none
- после запуска стоит проверить статус: minikube status
- запущенные служебные компоненты можно увидеть командой: kubectl get pods --namespace=kube-system

### Для сброса кластера стоит удалить кластер и создать заново:
- minikube delete
- minikube start --vm-driver=none

Возможно, для повторного запуска потребуется выполнить команду: sudo sysctl fs.protected_regular=0

Инструкция по установке Minikube - [ссылка](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)

**Важно**: t3.small не входит во free tier, следите за бюджетом аккаунта и удаляйте виртуалку.
</details>  

### Решение

- Выполнена установка в yandex cloud (развернуто через ansible, задание 4)

```bash
minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

```bash
kubectl get pods --namespace=kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-64897985d-h4724            1/1     Running   0          6m45s
etcd-minikube                      1/1     Running   0          6m57s
kube-apiserver-minikube            1/1     Running   0          6m59s
kube-controller-manager-minikube   1/1     Running   0          6m57s
kube-proxy-8bpz5                   1/1     Running   0          6m46s
kube-scheduler-minikube            1/1     Running   0          6m57s
storage-provisioner                1/1     Running   0          6m55s
```
---

## Задача 2: Запуск Hello World
<details>

  <summary>Описание задачи</summary>  

После установки Minikube требуется его проверить. Для этого подойдет стандартное приложение hello world. А для доступа к нему потребуется ingress.

- развернуть через Minikube тестовое приложение по [туториалу](https://kubernetes.io/ru/docs/tutorials/hello-minikube/#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BB%D0%B0%D1%81%D1%82%D0%B5%D1%80%D0%B0-minikube)
- установить аддоны ingress и dashboard
</details>  

### Решение

```bash
# Создаем deployment hello-world
kubectl create deployment hello-world --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/hello-node created

# Проверяем deployment hello-world
kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   1/1     1            1           5s

# Просматриваем созданные поды
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-world-bd79c8b9f-tnzw7   1/1     Running   0          75s

# Устанавливаем ingress и dashboard
minikube addons enable ingress
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled

minikube addons enable dashboard
    ▪ Using image kubernetesui/dashboard:v2.3.1
    ▪ Using image kubernetesui/metrics-scraper:v1.0.7
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

        minikube addons enable metrics-server
🌟  The 'dashboard' addon is enabled
```

---

## Задача 3: Установить kubectl
<details>

  <summary>Описание задачи</summary>  

Подготовить рабочую машину для управления корпоративным кластером. Установить клиентское приложение kubectl.
- подключиться к minikube 
- проверить работу приложения из задания 2, запустив port-forward до кластера
</details>  

### Решение

```bash
# создаем port-forward в hellopworld на 8080 порт
kubectl port-forward deployment/hello-world 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

```bash
# проверяем
curl localhost:8080
Hello, world!
Version: 1.0.0
Hostname: hello-world-deployment-7d7df99987-k7bq8
```


---

## Задача 4 (*): собрать через ansible (необязательное)
<details>

  <summary>Описание задачи</summary>  

Профессионалы не делают одну и ту же задачу два раза. Давайте закрепим полученные навыки, автоматизировав выполнение заданий  ansible-скриптами. При выполнении задания обратите внимание на доступные модули для k8s под ansible.
 - собрать роль для установки minikube на aws сервисе (с установкой ingress)
 - собрать роль для запуска в кластере hello world
</details>  

### Решение

1. Плейбук расположен ./ansible-minikube, комментарии в site.yml
2. Устанавливаем модуль kubernetes.core (добавлено в requirements.yml)
> ansible-galaxy collection install -r requirements.yml
3. Указываем имя хоста в ./ansible-minikube/inventory/hosts.yml
> ansible_host: <<***>>
4. Запускаем плейбук
> ansible-playbook -i ./inventory/ site.yml 
5. После выполнения проверям. На хосте с установленным minikube:

```bash
# Проверка ingress
curl hello-world.local
Hello, world!
Version: 1.0.0
Hostname: hello-world-deployment-7d7df99987-k7bq8

# deployments
get deployments
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-deployment   2/2     2            2           6m31s

# pods (два пода, потому что в деплойменте указал создание двух реплик)
kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
hello-world-deployment-7d7df99987-k7bq8   1/1     Running   0          7m14s
hello-world-deployment-7d7df99987-l7jg9   1/1     Running   0          7m14s

# ingress
kubectl get ingress
NAME                  CLASS   HOSTS               ADDRESS       PORTS   AGE
hello-world-ingress   nginx   hello-world.local   10.128.0.20   80      14m
```


- Устанавливать на хост с Ubuntu.
- В плейбуке возможны проблемы с идемпотентностью, сильно не усложнял. Приведено для примера.
- yml с deployment, service и ingress в папке ./ansible-minikube/templates
  

---
