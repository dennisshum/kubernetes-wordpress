# kubernetes-wordpress

## Docker

### create dockerfiles (wordpress and mariadb)
### build images
```
docker build . -f .\dockerfiles\wordpress.dockerfile -t dennisshum/wordpress
docker build . -f .\dockerfiles\mariadb.dockerfile -t dennisshum/mariadb
```
### create network
```
docker network create wordpress-network
```
### run images
```
docker run --rm -d --name mariadb --net wordpress-network `
-e MYSQL_ROOT_PASSWORD=my_password_1234789 `
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

