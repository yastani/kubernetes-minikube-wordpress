# kubernetes-minikube-wordpress
Minikubeを使用してローカル環境にWordpress+MySQLを構築する

## 開発環境構築手順

### 関連ツールのインストール
```bash
$ brew cask install minikube docker virtualbox kubernetes-cli kubectx
```

### MinikubeをVirtualboxで起動する
※minikube v0.26より `bootstrapper` のデフォルトが `localkube` から `kubeadm` に変更になった
```bash
minikube start \
  --vm-driver=virtualbox \
  --kubernetes-version=v1.11.10 \
  --bootstrapper=kubeadm \
  --memory=2048
```

### minikubeが起動していることを確認
こんな感じだったら成功
```bash
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.101
$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    <none>   2m54s   v1.14.2
```

#### Error starting host ~ が出たとき
http://kakts-tec.hatenablog.com/entry/2018/02/28/143338
>minikube delete してローカルのkubernetes clusterを削除した上で,minikube startしなおすと再度minikube vmを立ち上げれるようになる

### NySQLのパスワードをSecretに登録
※クラウド上のSecretは管理者が別途作成するので、この作業はローカル環境のみ
```bash
$ echo -n "mysql-pass" | base64
bXlzcWwtcGFzcw==
```

### MySQL Containerを作成
```bash
$ kubectl apply -f .local/mysql/manifests/
persistentvolumeclaim/mysql-pv-claim created
persistentvolume/mysql-pv-1 created
secret/mysql-secret created
deployment.apps/mysql created
service/mysql created
```

### MySQL ContainerのIPを確認
```bash
$ kubectl get pods -o wide -l app=dev
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
mysql-748c7777f6-8swxk   1/1     Running   0          4m11s   172.17.0.4   minikube   <none>           <none>
```

### MySQL ClientのDocker imageを一時的に作成してそこからMySQL Containerに接続
```bash
$ kubectl run -it --rm --image=mysql:5.7 --restart=Never mysql-client -- mysql -uroot -h 172.17.0.4 -pmysql-pass
If you don't see a command prompt, try pressing enter.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

### Wordpress Containerを作成
```bash
$ kubectl apply -f frontend/manifests/service.yml
service/wordpress configured
$ kubectl apply -f frontend/manifests/deployment.yml && kubectl rollout status -f frontend/manifests/deployment.yml
deployment.apps/wordpress created
Waiting for deployment "wordpress" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "wordpress" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "wordpress" rollout to finish: 2 of 3 updated replicas are available...
deployment "wordpress" successfully rolled out
```

### wordpressのpodが3つあることを確認
```bash
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
mysql-67cdf8766b-2gv9m       1/1     Running   1          3h16m
wordpress-5bdb9db4f7-bgz4x   1/1     Running   0          10m
wordpress-5bdb9db4f7-bkwcj   1/1     Running   0          43s
wordpress-5bdb9db4f7-xvbrm   1/1     Running   0          43s
```

### minikube serviceコマンドでブラウザからアクセス
Wordpressの初期セットアップ画面が表示されれば成功
```bash
$ minikube service wordpress
🎉  Opening kubernetes service default/wordpress in default browser...
```

## 参考

https://github.com/takaishi/hello2018/tree/master/k8s_hands_on