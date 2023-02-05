npm 知识清单
===

[npm](https://www.npmjs.com/)，常见知识点：镜像管理，安装、运行、打包、登录，发布等命令

常见命令
----
<!--rehype:body-class=cols-2-->

### 镜像管理
```bash
# 设置镜像
npm config set registry https://registry.npm.taobao.org
# nrm镜像管理工具
npm install -g nrm
# 原始镜像
nrm add npm https://registry.npmjs.org
# 国内镜像
anrm add taobao https://registry.npm.taobao.org
# 镜像切换
nrm use taobao
```

### 常见命令
```bash
# 安装
$ npm install
# 运行
$ npm run
# 编译
$ npm build
```

### 发布步骤
```bash
# 注册npm账号
https://www.npmjs.com/
# 切回原镜像
npm config set registry https://registry.npmjs.org
# 登录(输入账号、密码、邮箱)
npm origin
# 验证
npm whoami
# 发布
npm publish
# 更新
npm version patch/minor/marjor (v1.0.0-->版本号位数依次增大)
npm publish
```

npx
---
