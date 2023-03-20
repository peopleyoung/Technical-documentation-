# harbor使用教程

## 网页端

网址：https://harbor.zmjkf.cn/。账号:AIvision,密码：Zmj123456#。项目名称：aivision

## 服务器docker配置

### 1.配置域名

```
sudo vim /etc/hosts
#添加以下内容，前边为内网对应IP地址，后边为域名
172.16.66.223 harbor.zmjkf.cn
```

### 2.添加docker 信任

```
sudo vim /etc/docker/daemon.json
#若文件为空加入以下内容
{"insecure-registries": ["harbor.zmjkf.cn"]}
#若文件不为空则直接在大括号内加入:
"insecure-registries": ["harbor.zmjkf.cn"]
```

### 3.重启docker

```
sudo systemctl restart docker
```

## 上传镜像

### 1.登录

```
docker login harbor.zmjkf.cn
#需输入账号密码.账号:AIvision,密码：Zmj123456#
```

### 2.将已有docker名字标记为项目镜像

```
#harbor.zmjkf.cn/aivision为固定写法，代表推送到aivision项目，不可更改
docker tag SOURCE_IMAGE[:TAG] harbor.zmjkf.cn/aivision/REPOSITORY[:TAG]
```

### 3.推送镜像到项目

```
docker push harbor.zmjkf.cn/aivision/REPOSITORY[:TAG]
```

### 4.拉取镜像
、、、
docker pull harbor.zmjkf.cn/aivision/REPOSITORY[:TAG]
、、、
