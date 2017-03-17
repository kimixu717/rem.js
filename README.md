http://www.jianshu.com/p/985d26b40199
https://github.com/ant-design/ant-design-mobile/wiki/antd-mobile-0.8-%E4%BB%A5%E4%B8%8A%E7%89%88%E6%9C%AC%E3%80%8C%E9%AB%98%E6%B8%85%E3%80%8D%E6%96%B9%E6%A1%88%E8%AE%BE%E7%BD%AE

## 何为高清
高清就是在高清屏的手机上，`1px`实际显示的物理像素不为1，到时页面显示是会有1px的像素偏差，尤其是边框，比较影响页面的显示效果

* 用脚本设置 html 的 viewport （**不要再写 html meta 标签去设置 viewport**）：在 html > head 里
```html
<!DOCTYPE html>
<head>
 <meta charset="utf-8" />
 <title>title</title>
 <script>/** 高清方案脚本 */</script>
</head>
<body></body>
</html>
```
拷贝引入以下高清方案脚本，**请内联写到所有 css 引用之前, 否则部分安卓机有问题**（此脚本内部称为flex高清模式，支持任意等比缩放、兼容性好，其中原理请自己探索）：

```js
!function(e){function t(a){if(i[a])return i[a].exports;var n=i[a]={exports:{},id:a,loaded:!1};return e[a].call(n.exports,n,n.exports,t),n.loaded=!0,n.exports}var i={};return t.m=e,t.c=i,t.p="",t(0)}([function(e,t){"use strict";Object.defineProperty(t,"__esModule",{value:!0});var i=window;t["default"]=i.flex=function(e,t){var a=e||100,n=t||1,r=i.document,o=navigator.userAgent,d=o.match(/Android[\S\s]+AppleWebkit\/(\d{3})/i),l=o.match(/U3\/((\d+|\.){5,})/i),c=l&&parseInt(l[1].split(".").join(""),10)>=80,p=navigator.appVersion.match(/(iphone|ipad|ipod)/gi),s=i.devicePixelRatio||1;p||d&&d[1]>534||c||(s=1);var u=1/s,m=r.querySelector('meta[name="viewport"]');m||(m=r.createElement("meta"),m.setAttribute("name","viewport"),r.head.appendChild(m)),m.setAttribute("content","width=device-width,user-scalable=no,initial-scale="+u+",maximum-scale="+u+",minimum-scale="+u),r.documentElement.style.fontSize=a/2*s*n+"px"},e.exports=t["default"]}]);
flex(100, 1);
```
附未压缩的[源码链接](https://os.alipayobjects.com/rmsportal/XKeZwTvyfsCqhRfcVIAY.js)

* 『新项目忽略』在已有的未做类似高清方案的项目 less 文件中对 px 单位做 2 倍处理，可以使用 gulp 脚本（如果不是使用 gulp，则推荐使用 webpack，相应配置方法见下条），在 gulpfile.js 加入以下代码：

```js
var gulp = require('gulp');
var replace = require('gulp-replace');
var gulpif = require('gulp-if');
var exec = require('child_process').exec;
var srcFiles = [
  './src/**/*.less'
];
gulp.task('doublepx', function(done){
  gulp.src(srcFiles)
    .pipe(gulpif(true, replace(/['"](\d+)px['"]|\b(\d+)px\b/g, function(pixel) {
      console.log(pixel, '=>', ( parseInt(pixel) * 2 ) + 'px');
      if ( /'|"/.test(pixel) || '0px'== pixel || '1px' == pixel) {
        return pixel;
      }
      return ( parseInt(pixel) * 2 ) + 'px';
    })))
    .pipe(gulp.dest('./src'));
  done();
});
gulp.task('default', ['doublepx']);
```
* 随后新的样式值都写成和视觉稿上标注的值（视觉稿标注一般以设备物理点为单位，像 iPhone6 屏幕 750 的宽度）一样即可。
* 对于使用 webpack 的项目，在`webpack.config.js`里新增`pxtorem`配置、代码如下：

```js
  const pxtorem = require('postcss-pxtorem');
  webpackConfig.postcss.push(pxtorem({
    rootValue: 100,
    propWhiteList: [],
  }));
```

* 至此，高清方案设置完毕（过程中相关依赖请自行`npm install`）。

* 异常排查：通常以iPhone6模拟器为基准，查看`document.documentElement.clientWidth`（浏览器渲染出来的网页宽度），
  * 如果是750，则代表高清方案设置成功，接下来就是webpack的设置。
  * 如果是350，代表可能写死了meta标签，请先完成动态设置viewport这一步。
  * 如果是980左右，代表没有生成meta标签也没有写死meta标签，按照PC浏览器来渲染网页宽度。请先完成动态设置viewport这一步。