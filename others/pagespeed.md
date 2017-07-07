# PageSpeed

使用 ngx_pagespeed dynamic module

需要使用 Nginx 官方仓库中的 Nginx 版本

## 添加源

```
sudo yum install https://extras.getpagespeed.com/redhat/7/noarch/RPMS/getpagespeed-extras-7-0.el7.gps.noarch.rpm
```

## 安装

```
sudo yum install nginx-module-nps
```

## 配置

1. 加载模块

在 `/etc/nginx/nginx.conf` 开头加入：

```
load_module modules/ngx_pagespeed.so;
```

2. 启用

在需要启用 PageSpeed 的 `server` block 内加入：

```

pagespeed on;
pagespeed FileCachePath /tmp/ngx_pagespeed;
pagespeed EnableFilters recompress_images,convert_to_webp_lossless,inline_images,resize_images,lazyload_images,responsive_images,inline_css,inline_javascript,remove_comments;
# 图片压缩质量默认 85，易导致部分图片压缩后视觉上损失严重
pagespeed ImageRecompressionQuality 90;
```

默认 PageSpeed 只处理当前域名下的静态资源，如果使用了 CDN，可作如下配置：

```
# CDN 域名
pagespeed Domain assets.cdn.com;

# 如果 Nginx 用作反向代理，将 CDN 映射到所代理的地址，以便 Nginx 从所代理的地址获取静态资源而非从 CDN 获取
pagespeed MapOriginDomain 127.0.0.1:10090 assets.cdn.com;
```

## 参考资料

* https://www.getpagespeed.com/server-setup/nginx/install-ngx_pagespeed-dynamic-module-centos-7
* https://developers.google.com/speed/pagespeed/insights/
* https://www.modpagespeed.com/
* https://www.modpagespeed.com/doc/
