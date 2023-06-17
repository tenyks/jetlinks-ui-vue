<h1 align="center"> JetLinks-ui-vue</h1>

## 平台简介

* 本仓库为JetLinks vue版本。
* 前端技术栈[Vue3](https://v3.cn.vuejs.org) + [Jetlinks-ui-components](https://github.com/jetlinks/jetlinks-ui-components) + [Vite 4.x](https://cn.vitejs.dev)
* 本项目采用约定式路由，文件系统即路由，通过目录和文件及其命名分析出路由配置。

## 前端运行
```bash
# 克隆项目
git clone https://github.com/jetlinks/jetlinks-ui-vue.git

# 安装依赖
yarn

# 启动服务
yarn dev

# 更新jetlinks-ui-components
yarn add jetlinks-ui-components@1.0.10

# 更新jetlinks-ui-components之后没有效果时
yarn dev:force

# 打包
yarn build
```

### 备注

项目在开发模式下，首页加载慢属于正常现象；