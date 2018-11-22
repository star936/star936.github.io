### 基于Hexo+Github搭建的[个人博客](http://star936.github.io),地址为:http://star936.github.io

#### 依赖
* NodeJS
* NPM
* hexo

#### 本地运行
##### 1. 下载
```bash
git clone https://github.com/star936/star936.github.io.git
```
##### 2. 安装依赖
*确保已经安装Nodejs和npm*
在项目根目录下,执行如下命令:
```bash
npm install 
```
##### 3. 更换theme
在根目录的`themes`目录下面,执行如下命令:
```bash
git clone https://github.com/star936/hexo-theme-matery.git
```
修改`themes/hexo-theme-matery/_config.yml`,例如:
* **indexbtn的url**
* **livere的uid或者将enable改为false**
*其他信息自己根据需要更改.*

##### 4. 更换全局的配置
修改根目录下的`_config.yml`,例如:
* **title、author等基本信息**
* **有关GitHub的地址**
* **有关deploy的配置,需要配置git的SSH**

##### 5. 运行
```bash
# 生成静态文件
hexo g
# 本地启动
hexo s
# 部署
hexo d
```
