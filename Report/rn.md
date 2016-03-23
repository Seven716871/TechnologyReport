# reat navie 开发原生 app 的一些考虑

## 商业考虑：受制于人

1、三方库说砍就砍，fb 一停止更新此库，用这库的项目就只能干瞪眼

2、比如 iOS 系统每发一个新版本，有新功能或者 api 变动的，三方库可能一两月都不能支持，只能等

3、三方库的质量也堪忧，稳定成本高

4、团队人员流动的维护成本高，人才成本高


## 技术考虑：过渡非主流技术

1、从技术发展角度考虑，以后的技术主流是原生 app 内嵌网页的 hybrid 方式，就像淘宝，京东 app 现在这样

2、react native 之所以出现，也是有历史原因的：就是现有的内嵌网页在渲染性能上太差，于是才出现了用开发网页的技术开发原生 app，这样就解决了渲染性能问题

但是 react native 这种方案只能说是一种 app 发展过程由于内嵌网页方式性能不足的一个短期过渡技术

- 这个短期有多短：大家想像 xp 到 vista 到 win 7, 中 vista 的历史地位就懂了

- 这个过渡时期什么时候结束：内嵌网页性能足够就结束，可能也就是未来两三年的事，就连现在的淘宝 app 内嵌网页也能达到可观的性能水准

3、学习成本太高，非前端人员需要掌握：javascript，react，flex 等前端技术才能很好地入门。


## 目标受众：

- 产品性质：简单 GUI 数据展示型非商业级安全的项目，非常适合做所谓的能跨平台的外包 app。

- 开发人员：前端想玩 app 开发的


所以感觉时代的发展方向是 hybrid 内嵌网页，而不是 react native 这种临时过渡方案


## 关于 hybrid 方式：

- native 框架

- html5 + js

- js runtime 注入 native 功能

- 热更新
