# npm(node package manager)

## 简介
  NPM的全称是Node Package Manager，是随同NodeJS一起安装的包管理和分发工具，它很方便让JavaScript开发者下载、安装、上传以及管理已经安装的包。具体的细节可直接到[npm中文文档](https://www.npmjs.com.cn/).

## 1.npm使用随手记
### 1.1npm组成
- **网站** ：是开发者查找包（package）、设置参数以及管理 npm 使用体验的主要途径。
- **注册表(registry)** ：是一个巨大的数据库，保存了每个包（package）的信息。
- **命令行工具(CLI)** ：通过命令行或终端运行。开发者通过 CLI 与 npm 打交道。

### 1.2npm常用命令
&emsp; ```$ npm install packageName: 默认安装最新版本的包```
   
&emsp; ```$ npm install packageName@version: 安装指定版本的包```   

&emsp; ```$ npm install packageName -S  or $ npm install packageName --save: 安装包信息将放到package.json中的dependencies(生产阶段的依赖)```   

&emsp; ```$ npm install packageName -D  or $ npm install packageName --save-dev: 安装包信息将放到package.json中的devDependencies(开发阶段的依赖)，所以开发阶段一般使用它```   

&emsp; ```$ npm install packageName -E  or $ npm install packageName --save-exact: 精确安装指定版本。如：npm install express -ES,会发现dependencies中的express的版本号没有了^ 直接是4.0.0 。-ES实际是-E和-S，可组合使用```   

&emsp; ```$ npm install: 如果同目录下有对应的package.json文件，将根据package.json的依赖全部安装```  

&emsp; ```$ npm update packageName: 更新指定模块，更新完没有任何输出```   

&emsp; ```$ npm outdated: 此命令可以列出已经过时的包，发现过时的包之后，可以及时进行包的更新```   

### 1.3npm配置
基础语法：
```
npm config set <key> <value> [-g|--global]
npm config get <key>
npm config delete <key>
npm config list
npm config edit
npm get <key>
npm set <key> <value> [-g|--global]
```
1.对于config这块用得最多应该是设置代理，解决npm安装一些模块失败的问题.  
2.例如在公司内网，因为公司的防火墙原因，无法完成任何模块的安装，这个时候设置代理可以解决:npm config set proxy=http://www.proxy.com   
3.又如国内的网络环境问题，某官方的IP可能被和谐了，幸好国内有好心人，搭建了镜像，此时我们简单设置镜像: npm config set registry="http://r.cnpmjs.org"   
4.也可以临时配置，如安装淘宝镜像: npm install -g cnpm --registry=https://registry.npm.taobao.org 。有一个需要注意的地方，cnpm 在安装包时使用的是 cnpm/npminstall 。项目的 readme 已经写明了细节，仔细阅读即可。简单来说，npminstall 下的 node_modules 目录采用和 npm 官方 client 不一样的布局，主要是为了最大限度提高安装速度。如果既想安装速度快，又想保持和npm一样的目录结构，可使用npm config set registry="https://registry.npm.taobao.org" ，然后在使用npm安装依赖
