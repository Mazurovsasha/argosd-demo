## Инсталяция ArgoCD

[https://argo-cd.readthedocs.io/en/stable/getting_started/] Ссылка на оф.страницу с установкой

Установка

```bash
wget https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -O argocd-install.yaml
```

## Changes in argocd install:

```yaml
      containers:
      - args:
        - /usr/local/bin/argocd-server
        - --insecure     
        env:
        - name: ARGOCD_SERVER_INSECURE
```

## Add node port

```yaml
  name: argocd-server
spec:
  type: NodePort  ## Change service type
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 8080
    nodePort: 30007 ## Add port for bastion nginx
  - name: https
    port: 443
```

## Bastion nginx

```conf
server {
  listen 80;
  listen [::]:80;

  server_name "~^.*argocd\.k8s-(\d+)\.sa$";

  location / {
       # proxy_set_header Host $host;
        proxy_pass       http://192.168.203.$1:30007;
  }
}
```

Запускаем 

```bash
kubectl create namespace argocd  ### Создаем отдельный namespace

kubectl apply -f argocd-install.yaml -n argocd
```

## Веб интерфейс

1. Заходим в service > argocd-server > смотрим IP сервера

2. Закидывем IP в host на локальном компе (agrocd.k8s-7.sa)

3. Вытаскиваем пароль

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
либо заходим в admin secret, нажимаем "y" и копируем из password. Далее

```bash
echo "XXXXXX" | base64 -d
```
4. User: admin

## Довабляем источники

1. Setting > Repositories > CONNECT REPO

- Method: https

- Type: helm

- Repository URL: url репозитория с helm (на гите setting > Pages)

- Username (optional) and Password (optional) заполняются, если репозиторий приватный.

2. После добавления можно проверить в секретах на k8s.

3. Если надо добавить репо с yml файлом, а не с helm, то 

- Type: helm

## Applications

- SYNC POLICY: Automatic

- PRUNE RESOURCES Удаляет старые ресурсы, неиспользуемые. Очень полезная опция

## После доплоя

- APP DETAILS поменять опции

## Стащить yaml из kubernetes

```bash
kubectl get application first-app -n argocd -o yaml > first-app.yaml
kubectl get application jenkins -n argocd -o yaml > jenkins.yaml
kubectl get application nfs-subdir-external-provisioner -n argocd -o yaml > nfs-subdir-external-provisioner.yaml
```
