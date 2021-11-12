
## GitHub Pages

参考： [https://docsify.js.org/#/zh-cn/deploy](https://docsify.js.org/#/zh-cn/deploy)

使用`GitHub Pages`部署该文档项目非常简单，这里假定你已经有了`GitHub`账号，没有的话，注册一下。

- 第一步：`Fork`我当前`island`仓库，即 [https://github.com/bytesfly/island](https://github.com/bytesfly/island)

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211112104413154-992050666.png)

- 第二步：在刚`Fork`的仓库设置(`Settings`)页面开启`GitHub Pages`功能

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211112104758702-781100833.png)

然后，你就可以打开`https://<yourname>.github.io/island`看看效果了。


## 本地部署

> [!NOTE|label:Question]
> 如何在本地编辑文档并实时预览效果呢？

- 第一步： 克隆文档项目

仓库所在地址： [https://github.com/bytesfly/island](https://github.com/bytesfly/island)
```bash
git clone git@github.com:bytesfly/island.git
```

- 第二步： 安装启动nginx

`Linux`系统：
```bash
# 安装
sudo apt-get install nginx

# 查看状态
sudo systemctl status nginx

# 启动
sudo systemctl start nginx
```

`Windows`系统(待补充)：
```bash
# TODO
```
如果安装启动成功，浏览器打开 [http://localhost/](http://localhost/) ，可见如下界面：

![](images/nginx-20211105143122.jpg)

- 第三步： 配置nginx

`Linux`系统：

```bash
# 进入nginx配置目录
cd /etc/nginx/conf.d

# 创建新配置
sudo touch island.conf
```
然后打开`island.conf`，添加如下内容：
```text
server {
  listen 12345;
  root /home/bytesfly/proj/island;
  index index.html;
  location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    root /home/bytesfly/proj/island;
  }
}
```
其中`root`后面配置的是刚才克隆的`island`项目绝对路径。

再执行命令让`nginx`重新加载：
```bash
sudo nginx -s reload
```
浏览器打开 [http://localhost:12345/](http://localhost:12345/) ,如下

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211112100936828-238827122.png)

> [!WARNING]
> 此时，用你喜欢的本地编辑器编写`Markdown`文档并保存，浏览器刷新页面(`Ctrl + F5`)即可实时预览效果。

## 补充Docker部署

当然，如果本地有Docker环境，也可使用Docker部署。下面以`docker-compose`为例。

下面是整体目录结构，当前目录下有`docker-compose.yml`文件和`conf.d`文件夹，`conf.d`文件夹下有`island.conf`文件。

```bash
➜  ~ tree
.
├── conf.d
│   └── island.conf
└── docker-compose.yml

1 directory, 2 files
```

`docker-compose.yml`文件内容如下：

```yaml
version: '3.9'
services:
  nginx:
    image: nginx:1.20.1
    volumes:
      - ./conf.d:/etc/nginx/conf.d:ro
      - /home/bytesfly/proj/island:/var/www
    ports:
      - "8080:8080"
    networks:
      internal:  {}
    restart: always
networks:
  internal: {}
```

其中`/home/bytesfly/proj/island`是文档项目所在绝对路径。

`island.conf`文件内容如下：

```text
server {
  listen 8080;
  root /var/www;
  index index.html;
  location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt){
    root /var/www;
  }
}
```

在`docker-compose.yml`当前目录执行如下命令：

```bash
sudo docker-compose up -d
```

如果没有其他问题的话，浏览器打开 [http://localhost:8080/](http://localhost:8080/) 查看文档。