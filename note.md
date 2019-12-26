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
