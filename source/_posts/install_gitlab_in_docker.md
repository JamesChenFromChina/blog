title: 使用Docker安装GitLab
date: 2017-10-17
tags: gitlab docker
categories: tools
---

最近公司服务器下架，准备在新分配的服务器上面部署gitlab，之前对Gitlab的安装一直抱有小阴影，因为依赖的各种工具太多了，而且可能也会影响到服务器未来的使用，比如部署其他的服务需要使用redis等等情况吧，然而惊喜的发现了gitlab现在有了基于Docker的安装方式，所以决定尝试一下，因此有了本篇博文。


按照[官方套路](https://docs.gitlab.com/ce/install/docker.html) 找到了Docker的gitlab image, [官方文档](https://docs.gitlab.com/omnibus/docker/)在此

很官方的执行了 docker pull gitlab/gitlab-ce 拉取了gitlab的最新镜像

然后准备 
```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

这里由于本机已经运行了sshd服务，所以 22端口已经被占用了，所以修改了命令

```
sudo docker run --detach --hostname 172.16.10.81 --publish 443:443 --publish 80:80 --publish 2222:22 --name gitlab --restart always --volume /srv/gitlab/config:/etc/gitlab --volume /srv/gitlab/logs:/var/log/gitlab --volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:latest
```

正常启动运行创建账号都OK 但是创建项目的时候发现项目中默认的拉去指示代码还是基于默认ssh端口的，之后准备更改服务端口
`sudo docker exec -it gitlab /bin/bash` 连接上去，对gitlab的配置文件进行修改

`root@172:/# vim /etc/gitlab/gitlab.rb`  在里面找到了ssh的端口配置项 `gitlab_rails['gitlab_shell_ssh_port'] = 2222` ，修改成了2222，并且docker rm gitlab删除了之前的容器，使用


```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:2222 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

```

构建新容器。 docker restart gitlab重启， 然后发现2222端口没有动静,最后发现这个镜像的sshd服务配置文件在 `/assets/sshd_config` 这个位置，修改了sshd的监听端口后搞定
