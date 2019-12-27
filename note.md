## NUXT项目打包之后js文件太大，解决方案
* 1、在nuxt.config文件中
```
extend(config, {isDev}) {
      if (isDev) {
        config.devtool = '#source-map'
      }
      /**
       * upload-to-ali组件依赖ali-oss脚本，体积较大。
       * 这里将该依赖放在script处用引入，可利用cdn加速，并减少项目最终打包体积
       * FYI: 如果不需要upload-to-ali组件，记得在移除组件后也要移除在script引用的ali-oss脚本
       */
      config.externals = {
        'ali-oss': 'OSS',
      }
    },
    vendor: ['element-ui'],
    maxChunkSize: 300000,
    // 开启打包分析(重点，可以找到文件大的原因stat，parsed，gzipped)
    // analyze: true,
    // assetFilter: function(assetFilename) {
    //   return assetFilename.endsWith('.js');
    // }
  },
```
* 2、在nginx配置gizp
```
        gzip on;
        gzip_disable "msie6";

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 1;
        gzip_buffers 16 8k;
        gzip_http_version 1.0;
        gzip_min_length 256;
        gzip_types text/plain text/css
                   application/json application/x-javascript text/xml
                   application/xml application/xml+rss text/javascript application/javascript
                   application/vnd.ms-fontobject application/x-font-ttf font/opentype 				    image/svg+xml image/x-icon
                   image/jpeg image/gif image/png;

```
* 3、重启nginx，在浏览器中Response Headers看到 Content-Encoding:gzip;

## note
* 1、找到如下一段，进行修改

```gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
#gzip_http_version 1.0;
gzip_comp_level 2;
gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
gzip_vary off;
gzip_disable "MSIE [1-6]\.";
```

* 2、解释一下

第1行：开启Gzip

第2行：不压缩临界值，大于1K的才压缩，一般不用改

第3行：buffer，就是，嗯，算了不解释了，不用改

第4行：用了反向代理的话，末端通信是HTTP/1.0，有需求的应该也不用看我这科普文了；有这句的话注释了就行了，默认是HTTP/1.1

第5行：压缩级别，1-10，数字越大压缩的越好，时间也越长，看心情随便改吧

第6行：进行压缩的文件类型，缺啥补啥就行了，JavaScript有两种写法，最好都写上吧，总有人抱怨js文件没有压缩，其实多写一种格式就行了

第7行：跟Squid等缓存服务有关，on的话会在Header里增加"Vary: Accept-Encoding"，我不需要这玩意，自己对照情况看着办吧

第8行：IE6对Gzip不怎么友好，不给它Gzip了
