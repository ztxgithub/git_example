[scloud](http://sc.kongtrolink.com/index) — 蓄电池监控系统
==================================================

--------------------------------------
> # 项目开发环境

- 硬件环境
 - 操作系统：Windows 7及以上 / Linux / Mac OS X等
 - 硬件环境：PC
 - 平台环境：浏览器支持（IE 10及以上 / Chrome / Firefox / Safari等）

- 软件开发环境
 - 软件开发环境前后端分离


--------------------------------------
> ## Scloud前端

> #### 开发配置 

- [NodeJS（需要安装4.5稳定版本及以上）](http://nodejs.cn/)
- [前端IDE（可选则）](https://www.jetbrains.com/webstorm/)
- [浏览器调试（建议Chrome）](https://www.google.com/chrome/browser/desktop/index.html)
- [git项目管理工具](https://git-scm.com/)
- [npm包管理工具淘宝国内镜像](https://npm.taobao.org/)



> #### 运行前端项目

- 打开命令行工具，通过运行以下命令来克隆Scould项目副本

```bash
git clone http://git.kongtrolink.com:3000/yytd/ScloudWeb.git
```

- 若无VPN，可以使用淘宝镜像下载依赖(之后可用cnpm代替npm)

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

- 进入项目下ScloudWeb/src/main/node目录下,安装所有依赖

```bash
cd ScloudWeb/src/main/node && npm install
```

- 项目本地运行（使用本地mock数据）

```bash
gulp serve
```

- gulp serve 有以下几个参数：
 - --dev 开启服务前跳过执行less编译。否则会在开启服务前执行less编译。
 - --f  使用全局代理，这个情况下，服务只使用代理服务器传递的数据。否则会优先使用本地的mock数据。
 - --u  指定代理服务器的IP（如 --u 10.0.5.99）。否则使用mock/config.json下的domain。
 - --p  指定代理服务器的端口号 (如 --p 8080)。否则使用mock/config.json下的port。
 - --dist 使用打包发布的项目文件。否则使用webapp下res目录内的项目文件。
 
```bash
gulp serve --dev --dist --f --u 10.0.5.99 --p 8080
```

- 项目打包发布（目标文件生成于dist目录下）

```bash
gulp bulid || gulp
```

- 注: 项目运行详情可查看node目录下gulpfile.json配置文件



> #### 运行单元测试

- 在ScloudWeb/src/main/node目录下，命令行运行

```bash
cd ScloudWeb/src/main/node && karma start karma.conf.js
```

- 测试用例默认运行在chrome浏览器，若需用其他浏览器运行可在karma.conf.js配置

```javascript
	//修改browsers配置项即可
	browsers: ['Chrome'],
```


> #### 查看测试覆盖详情

- 打开kama.conf.js配置文件同层目录 coverage/Chrome（XXXXXXXXXXXXXX）/index.html 网页文件查看即可
