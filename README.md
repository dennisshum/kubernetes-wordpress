# kubernetes-wordpress

## Docker

### create dockerfiles (wordpress and mariadb)
### build images
```
docker build . -f .\dockerfiles\wordpress.dockerfile -t dennisshum/wordpress
```
```
docker build . -f .\dockerfiles\mariadb.dockerfile -t dennisshum/mariadb
```
### create network
```
docker network create wordpress-network
```
### run images
```
docker run --rm -d --name mariadb --net wordpress-network `
-e MYSQL_RANDOM_ROOT_PASSWORD=1 `
-e MYSQL_DATABASE=my_wp_database `
-e MYSQL_USER=my_wp_user `
-e MYSQL_PASSWORD=my_wp_user_password `
-v ${PWD}/mysql:/var/lib/mysql `
dennisshum/mariadb
```
```
docker run --rm -d --name wordpress -p 80:80 --net wordpress-network `
-e WORDPRESS_DB_HOST=mariadb `
-e WORDPRESS_DB_USER=my_wp_user `
-e WORDPRESS_DB_PASSWORD=my_wp_user_password `
-e WORDPRESS_DB_NAME=my_wp_database `
dennisshum/wordpress
```
### Clean up
```
docker ps
docker rm -f wordpress
docker rm -f mariadb
docker network rm wordpress-network
rm mysql
```

## Kubernetes

### Install kind
https://kind.sigs.k8s.io/

### Create cluster
```
kind create cluster --name testing
```
check cluster
```
kubectl config get-contexts
```

### Create namespace
```
kubectl create ns wp-ns
```
check namespace
```
kubectl get ns
```

### Create configmap
```
kubectl -n wp-ns create cm mariadb `
--from-literal MYSQL_RANDOM_ROOT_PASSWORD=1
```
or
```
kubectl -n wp-ns create cm mariadb `
--from-env-file=env/configmap/mariadb.env
```
check configmap
```
kubectl -n wp-ns get cm mariadb -o yaml
```

### Create secret
```
kubectl -n wp-ns create secret generic wordpress `
--from-env-file=env/secret/wordpress.env
```
```
kubectl -n wp-ns create secret generic mariadb `
--from-env-file=env/secret/mariadb.env
```
check secret
```
kubectl -n wp-ns get secret wordpress -o yaml
kubectl -n wp-ns get secret mariadb -o yaml
```

### Deployment
```
kubectl -n wp-ns apply -f yaml/deployment.yaml
```
check deployment
```
kubectl -n wp-ns get deploy
kubectl -n wp-ns describe deploy wordpress-deployment
```

### Service
```
kubectl -n wp-ns apply -f yaml/service.yaml
```
check service
```
kubectl -n wp-ns get svc
kubectl -n wp-ns describe svc wordpress
```

### Check connection
```
kubectl -n wp-ns port-forward svc/wordpress 80
```

### Add Adminer service
```
kubectl -n wp-ns apply -f yaml/adminer-deployment.yaml
```
```
kubectl -n wp-ns apply -f yaml/adminer-service.yaml
```

### Check Adminer connection
```
kubectl -n wp-ns port-forward svc/adminer 8000
```

### Install Ingress
https://github.com/kubernetes/ingress-nginx

https://kubernetes.github.io/ingress-nginx/deploy/

check k8s version
```
kubectl version
```
install Controller according to K8S version
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
```
check ingress pod
```
kubectl -n ingress-nginx get pod
```
check nginx
```
kubectl -n ingress-nginx port-forward svc/ingress-nginx-controller 80
```

### Create Ingress
```
kubectl -n wp-ns apply -f yaml/ingress.yaml
```

### Clean up
```
kind delete cluster --name testing
```