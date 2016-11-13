---
title: wordpress 博客全站https
tags: [wordpress, https, let's encrypt]
categories: [wordpress]
---

### 为什么要用https
为了防止中间人攻击, 为了右下角不出现黄色广告, 为了带来更多的读者..., 网络环境越来越复杂, 各种运营商劫持已经使得我们不得不使用https来保障数据的安全. 公司网站已经全部https化, 我们个人博客亦不能拉下.
<!--more-->
### let's encrypt
要加密, 那就得有证书, [let's encrypt](https://letsencrypt.org/) 基本上是免费证书里面做得最好的了, 并且得到了chrome, mozilla, ie等主流浏览器的认可. let's encrypt 的自动续签功能, 只要一次配置好之后, 以后都不用担心过期的问题了.


#### 生成证书
[certbot](https://certbot.eff.org)提供了一套脚本帮助我们生成证书, 并且支持多域名, 这里简单的演示一下, 详细的请访问[官网](https://certbot.eff.org)

```
certbot certonly --standalone -d example.com -d www.example.com
```

#### 自动续签
```
certbot renew --dry-run
```

### 修改nginx配置
首先80的端口的http访问应该全部转成https
```
server {                                               
    listen       80;                                   
    server_name example.com www.example.com;               
    #charset koi8-r;                                   
    if ($server_port = 80) {                           
         rewrite ^http://$host https://$host permanent;
         rewrite ^(.*)$ https://$host$1 permanent;     

    }                                                  
}                                                      
```

打开443端口, 并加入证书
```
server {                                                                      
    listen       443 ssl;                                                     

    server_name example.com www.example.com;                                                    
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;        
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;          
    #charset koi8-r;                                                          

    #access_log  logs/host.access.log  main;                                  
    root /var/www/wordpress/;                                                 
    index index.php index.html;                                               

    location / {                                                              
        # try_files $uri $uri/ =404;                                          
        try_files $uri $uri/ /index.php?q=$uri&$args;                         
    }                                                                         

    location ~ \.php$ {                                                       
        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;                    
        fastcgi_index  index.php;                                             
        fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;  
        include        fastcgi_params;                                        
    }                                                                         
}                                                                             
```

证书生成之后的目录在/etc/letsencrypt/live/your-domain/

重启nginx, 如果没有意外我们的网站就应该有绿色小图标了.

![](https://og3qyrp94.qnssl.com/91a903c2b816add7cfac1c74917c0885.png)

### 修改网站链接
设置里面的网站链接, 要改成https的, 否则一些静态资源依然会是http.

![](https://og3qyrp94.qnssl.com/db624a667a8f3aeb8c7d1ba67bdb9176.png)

### 七牛服务转https
一般为了加速, 我们都会使用七牛的静态资源云存储, 如果使用了, 记得要改为使用https地址.

### Gravatar https
Gravatar 目测是使用http的链接, 要改为https的链接得使用一个插件WP Gravatar Https.

### 一些文章的图片
如果一些文章里面使用了http链接的图片, 得自己一个个去改了, 不过这个问题不大.

### 总结
目测这样一番改动之后, 整个博客就都是https的链接了, 小绿图标看着真开心!

Good Luck!
