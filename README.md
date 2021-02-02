从零使用 Webpack5 搭建一个完整的 Vue3 的开发环境
前言
为什么要造轮子？官方的脚手架不香吗？ 造轮子只是为了加深我们对技术的理解，在会用的基础上了解其后背的原理知识，这样才能在各种业务需求里灵活的运用其知识点，达到事半功倍的效果~

下面是公司搭建的真实项目开发环境我提炼出来比较重要的一些知识点作为内部技术分享。

接下来，我们使用 Webpack5 搭建一个完整的 Vue3 的开发环境！！！


初始化目录
第一步：初始化 package.json

npm init -y

第二步：安装 webpack

npm install webpack webpack-cli -D

注意：

-D 等价于 --save-dev; 开发环境时所需依赖
-S 等价于 --save; 生产环境时所需依赖
第三步：初始化目录和文件

创建 webpack.config.js 文件用于编写 webpack 配置

// webpack.config.js
const path = require('path');
  
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist')
    }
}

创建 src 目录用于存放源代码

// src/index.js
console.log('test webpack')

修改 webpack.config.js 中 scripts 字段

{
  // ...
  "scripts": {
    "build": "webpack"
  },
  // ....
}
  

打包

在项目根目录终端输入：npm run build 即可！

打包成功后，会在项目的根目录自动创建一个 dist 文件夹，里面的 main.js 文件就是我们打包后的文件。

配置核心功能
将 ES6+ 转 ES5
由于有些浏览器无法解析 ES6+ 等高级语法，故需要将其转化为浏览器能够解析的低级语法（如：IE浏览器）

第一步：安装依赖

npm install @babel/core babel-loader @babel/preset-env -D

第二步：修改 webpack.config.js 配置

const path = require('path');

module.exports = {
    // ...
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
        ]
    }
}

注意：若不想将配置写在配置文件中，可在项目根目录创建 babel.config.js 或 babelrc.js 文件。

处理样式
由于 webpack 默认只能打包处理 commonJs 规范的 js 文件，处理其他文件都需要相对应的处理器进行处理。

第一步：安装依赖

npm install style-loader css-loader less less-loader -D

第二步：修改 webpack.config.js 配置

const path = require('path');

module.exports = {
    // ...
    module: {
        rules: [
            // ...
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            }
        ]
    }
}

注意：loader 的配置有很多优化的地方，放在后面【优化】详细讲解。

处理图片等静态资源
同理，除 js 文件的其他文件打包 webpack 都需特定的处理器进行处理。

第一步：安装依赖

npm install url-loader file-loader -D

第二步：修改 webpack.config.js 配置

const path = require('path');

module.exports = {
    // ...
    module: {
        rules: [
            // ...
            {
                test: /\.(jpg|png|jpeg|gif|bmp)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        limit: 1024,
                        fallback: {
                            loader: 'file-loader',
                            options: {
                                name: '[name].[ext]'
                            }
                        }
                    }
                }
            },
            {
                test: /\.(mp4|ogg|mp3|wav)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        limit: 1024,
                        fallback: {
                            loader: 'file-loader',
                            options: {
                                name: '[name].[ext]'
                            }
                        }
                    }
                }
            }
        ]
    }
}

创建 html 文件
我们如何让打包的 js 文件自动的插入到html模板中呢？

第一步：安装依赖

npm install html-webpack-plugin -D

第二步：修改 webpack.config.js 配置

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    // ...
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            title: 'Vue3 + TS -> Web App'
        })
    ]
}

注意：配置动态网页标题时，需将模板中的 <title> 标签里的内容改成 <%= htmlWebpackPlugin.options.title %>

开发服务器
每次打包后都需要手动的点击生成的 index.html 看效果，可不可以让 webpack 将打包后的文件自动在浏览器打开呢？

第一步：安装依赖

npm install webpack-dev-server -D

第二步：修改 webpack.config.js 配置

module.exports = {
    // ...
    devServer: {
        port: 3000,
        hot: true,
        open: true,
        contentBase: '../dist'
    },
    // ...
}

清除打包文件
若打包的文件加了 hash，那每次打包生成的文件都会 dist 目录保留，我们可以使用此插件帮助我们每次打包前先清除以前的打包文件。

第一步：安装依赖

npm install clean-webpack-plugin -D

第二步：修改 webpack.config.js 配置

const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    // ...
    plugins: [
        // ...
        new CleanWebpackPlugin()
    ]
}

设置环境变量
设置环境变量有以下几种常见方式：

命令式
配置式
创建 .env 文件
cross-env
我们以 cross-env 的方式来设置环境变量, 因为他可以跨终端进行设置

第一步：安装依赖

npm install cross-env -D

第二步：修改 package.json 配置

{
    // ...
    "scripts": {
        "webpack": "cross-env NODE_ENV=development webpack"
    }
    // ...
}

分环境打包
在我们平时项目开发中，一般都会有：开发环境、测试环境和生产环境。什么样的打包文件适合于开发环境？适合于测试环境和生产环境？带着这些疑问我们来配置一个多环境打包。

打包压缩
我们开发一个项目的最终要求就是要达到功能完备的情况下打包体积最小才是我们想要的效果，因为这样可以很大程度上提供用户体验。

压缩 html 文件

修改 webpack.config.js 配置

const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    // ...
    plugins: [
        new HtmlWebpackPlugin({
            // ...
+            minify: {
+                collapseWhitespace: true, // 去掉空格
+                removeComments: true // 去掉注释
+            }
        }),
        // ...
    ]
}

压缩 css 文件

第一步：安装依赖

npm install mini-css-extract-plugin optimize-css-assets-webpack-plugin -D

第二步：修改 webpack.config.js 文件

const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
  
module.exports = {
    // ...
    module: {
        rules: [
            // ...
            {
                test: /\.css$/,
                use: [
+                   MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
+ 					MiniCssExtractPlugin.loader,
                    'css-loader',
                    'less-loader'
                ]
            },
            // ...
        ]
    },
    plugins: [
        // ...
        new OptimizeCssAssetsWebpackPlugin(),
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
        })
    ]
}

注意：purgecss-webpack-plugin 是用于清除无用的 CSS

压缩 js 文件

第一步：安装依赖

npm install terser-webpack-plugin -D

第二步：修改 webpack.config.js 文件

const TerserWebpackPlugin = require('terser-webpack-plugin');
  
module.exports = {
    // ...
    optimization: {
        minimize: true,
        minimizer: [
            new TerserWebpackPlugin()
        ]
    },
    // ...
}

注意：uglifyjs-webpack-plugin 不支持压缩 ES6 语法的代码

压缩图片

第一步：安装依赖

npm install image-webpack-loader -D

第二步：修改 webpack.config.js 文件

module.exports = {
    // ...
    module: {
        rules: [
            // ...
            {
                test: /\.(jpg|png|jpeg|gif|bmp)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 1024,
                            fallback: {
                                loader: 'file-loader',
                                options: {
                                    name: '[name].[ext]'
                                }
                            }
                        }
                    },
                    {
                        loader: 'image-webpack-loader',
                        options: {
                            mozjpeg: {
                                progressive: true,
                            },
                            optipng: {
                                enabled: false,
                            },
                            pngquant: {
                                quality: [0.65, 0.90],
                                speed: 4
                            },
                            gifsicle: {
                                interlaced: false,
                            },
                            webp: {
                                quality: 75
                            }
                        }
                    }
                ]
            },
            // ...
        ]
    },
    // ...
}

注意：在安装 image-webpack-loader 依赖时，采用 cnpm 安装。采用 npm 安装会报错：

Module build failed (from ./node_modules/image-webpack-loader/index.js):
Error: Cannot find module 'gifsicle'

集成 TypeScript
TypeScript 是目前前端工程师的必备技能，Vue3 的源码全部采用 TS 进行重写。故一定要掌握好并会灵活应用在真实的项目中。里面的一些核心概念要重点掌握：泛型、枚举、接口、类、函数等等

配置环境
第一步：安装依赖

npm install typescript ts-loader -D

第二步：修改 webpack.config.js 文件

module.exports = {
    // ...
    module: {
        rules: [
            {
                test: /\.ts$/,
                use: [
                    'ts-loader'
                ]
            },
            // ...
        ]
    },
    // ...
}

第三步：初始化 tsconfig.json 文件

tsc --init

重点内容
泛型
接口
函数
...
识别 .vue文件
第一步：安装依赖

npm install vue@next -S
npm install vue-loader@next @vue/compiler-sfc

注意：Vue2.x 时安装的是 vue-template-complier

第二步：修改 webpack.config.js 文件

const { VueLoaderPlugin } = require('vue-loader/dist/index');

module.exports = {
    // ...
    module: {
        rules: [
            {
                test: /\.vue$/,
                use: [
                    'vue-loader'
                ]
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin()
    ]
}

第三步：index.js 文件中引入 Vue

// index.js
import { createApp } from 'vue';
import App from './App.vue';

createApp(App).mount('#app')

新增 App.vue 文件

// App.vue
<template>
    <div>
        <div>解析Vue文件了哟~</div>
        <p>{{name}}</p>
    </div>
</template>
<script lang="ts">
import { defineComponent, ref } from 'vue';
export default defineComponent({
    setup() {
        const name = ref('txm')

        return {
            name
        }
    }
})
</script>

注意：defineComponent 只是为了在使用 Vue3 时有很好的语法提示

Composition API
Vue3 受 React Hooks 的启发，将以前的 options API 改写成 函数式API ，这样在很大程度上将代码进行解耦也便于 tree-shaking，提高代码的复用率。虽说 Vue2 中的 Mixin 可以完成公共逻辑代码的抽离，但是 Mixin 也有以下缺点：命名冲突、同名的方法和计算属性会被覆盖、同名的生命周期都会执行且 mixin 里的先执行等等。

比较常用的 Composition API 有：

reactive、ref、effect、watch、computed、生命周期、h函数、toRefs等等不一一列举
想更详细的了解请参考阅读 Vue 组合式 API

响应式系统
都知道 Vue2.x 的响应式底层核心是采用 Object.defineProperty 来劫持对象的每个属性的 getter和setter，在获取属性时做 依赖收集, 在更新属性时 触发更新。

Vue2.x 中响应式和方法: defineReactive

// src\core\observer\index.js

if (Array.isArray(value)) {
   this.observeArray(value)
} else {
   this.walk(value)
}
// 处理对象
walk (obj: Object) {
   const keys = Object.keys(obj)
   for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
   }
}
// 处理数组
observeArray (items: Array<any>) {
   for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
   }
}

export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend() // 收集依赖
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify() // 通知更新
    }
  })
}

从以上核心原理可以看出在处理 数组 和 对象 时进行判断分别进行处理。若数据是多层结构时一上来就进行 递归 操作并且是对对象的每个属性进行劫持对性能不好。

总结 Vue2.x 响应式层面的几个缺陷：
对象属性的 新增 和 删除 无法检测 -> 解决方法：Vue.$set 和 Vue.delete()
修改数组 索引 和 length 属性无法检测 -> 解决方法：splice
Vue3 则是采用 Proxy 作为底层响应式的核心 API，源码如下：

// packages\reactivity\src\reactive.ts

function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  return proxy
}

createReactiveObject 方法是创建响应式的核心方法，可以看出 Proxy 直接监听整个对象也没有对数组和对象进行分开处理。

注意：在学习 Vue3 源码时，需提前掌握好 Set、WeakSet、Map、WeakMap、Reflect等 ES6 新特性的语法.

新增特性
Vue3 也跟给开发者提供了几个新的内置组件：Fragment 、Suspense 、Teleport

集成 Vue-Router
基本使用
第一步：安装依赖

npm install vue-router@4 -S

第二步：创建 Home.vue 和 Me.vue文件

// Home.vue
<template>
    <div>
        首页
    </div>
</template>
<script>
import { defineComponent, ref } from 'vue';
export default defineComponent({
    name: 'Home'
})
</script>


// Me.vue
<template>
    <div>
        我的页面
    </div>
</template>
<script>
import { defineComponent, ref } from 'vue';
export default defineComponent({
    name: 'Me'
})
</script>

第三步：创建 router.js 文件

import { createRouter, createWebHistory } from 'vue-router';
import Home from './Home.vue';
import Me from './Me.vue';

const routerHistory = createWebHistory();

const router = createRouter({
    history: routerHistory,
    routes: [
        {
            path: '/home',
            name: 'Home',
            component: Home
        },
        {
            path: '/me',
            name: 'Me',
            component: Me
        }
    ]
})

export default router;

第四步：修改 index.js 文件

// index.js
import { createApp } from 'vue';
import App from './App.vue';
+ import router from './router.js';

createApp(App).use(router).mount('#app')

可以看到和原先 Vue2.x 中的路由用法还是有点区别，全部采用函数的方式了

集成 Vuex
基本使用
第一步：安装依赖

npm install vuex@next -S

第二步：创建 store.js 文件

import { createStore } from 'vuex';

const store = createStore({
    state: {
        name: 'vuex'
    },
    getters: {},
    actions: {},
    mutations: {},
    modules: {}
})

export default store;

第三步：修改 index.js 文件

import { createApp } from 'vue';
import App from './App.vue';
import router from './router.js';
+ import store from './store.js';

createApp(App).use(router).use(store).mount('#app')

第四步：在 App.vue 中获取 vuex 中的数据

<template>
    <div>
        <!-- ... -->
        <p>获取vuex里面的数据{{count}}</p>
        <!-- ... -->
    </div>
</template>
<script>
import { defineComponent, computed } from 'vue';
import { useStore } from 'vuex';
export default defineComponent({
    setup() {
        const store = useStore();
        const count = computed(() => store.state.count)

        return {
            count
        }
    }
})
</script>

注意：用 computed 包裹从 store 中获取的数据，可以保证数据的响应式

若不熟悉 Vue3 基本用法的小伙伴, 可参考阅读下这个基本入门 快速掌握 Vue3 全家桶开发

集成 Vant
第一步：安装依赖

npm install vant@next -S

按需引入
第一步：安装依赖

npm i babel-plugin-import ts-import-plugin -D

第二步：修改配置

JS版本

// babel.config.js
module.exports = {
  plugins: [
    [
      'import',
      {
        libraryName: 'vant',
        libraryDirectory: 'es',
        style: true,
      },
      'vant',
    ],
  ]
};


TS版本

const tsImportPluginFactory = require('ts-import-plugin');
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              transpileOnly: true,
              getCustomTransformers: () => ({
                before: [
                  tsImportPluginFactory({
                    libraryName: 'vant',
                    libraryDirectory: 'es',
                    style: (name) => `${name}/style/less`,
                  }),
                ],
              }),
              compilerOptions: {
                module: 'es2015',
              },
            }
          },
        ],
        exclude: /node_modules/
      },
      // ...
    ],
  }
};

Rem 布局适配
第一步：安装依赖

npm install lib-flexible -S
npm install postcss-pxtorem -D

第二步：添加 .postcssrc.js 文件

module.exports = {
    plugins:{
        // autoprefixer: {
        //     browsers: ['Android >= 4.0', 'iOS >= 8']
        // },
        'postcss-pxtorem': {
            // rootValue: 37.5, // Vant 官方根字体大小是 37.5
            rootValue({file}) {
                return file.indexOf('vant') !== -1 ? 37.5 : 75
            },
            propList: ['*'],
            selectorBlackList: ['.norem'] // 过滤掉.norem-开头的class，不进行rem转换
        }
    },
}

注意：browsers 选项需配置在 package.json，不然打包会有警告

// package.json
{
  // ...
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "Android >= 4.0",
    "iOS >= 8"
  ]
}


第三步：引入和使用

// index.js
import { createApp } from 'vue';
+ import 'lib-flexible/flexible';
import App from './App.vue';
import router from './router.js';
import store from './store.js';

createApp(App).use(router).use(store).mount('#app')

// Home.vue
<template>
  <div>
    首页
    <v-button plain hairline type="primary">细边框按钮</v-button>
    <v-button plain hairline type="primary">细边框按钮</v-button>
  </div>
</template>
<script>
import { defineComponent, ref } from "vue";
import { Button } from "vant";
export default defineComponent({
  name: "Home",
  components: {
    "v-button": Button
  }
});
</script>

优化
优化一般都是项目的重点部分，好的优化手段能让项目保证功能完整的情况下打包体积大大缩小，下面是几种项目实际开发中常用的优化手段：



规范目录结构
上面使用 Webpack5 将各个环境配置好后，规范化一下项目的目录结构, 规范后的项目目录结构如下：

tree -I "node_modules"

├─dist
│  ├─css
│  └─js
|  |-favicon.ico
|  |-index.html
├─node_modules
├─public
|  |-index.html
|  |-favicon.ico
└─src
|  ├─api
|  ├─components
|  ├─hooks
|  ├─router
|  ├─store
|  ├─utils
|  └─views
|  |-App.vue
|  |-main.ts
|-.gitigore
|-babel.config.js
|-package.json
|-shims-vue.d.ts
|-tsconfig.json
|-webpack.config.js

看这个目录结构是否有点像 Vue 脚手架生成的项目的目录

注意：由于 TypeScript 只能理解 .ts 文件，无法理解 .vue文件，故需在项目根目录创建一个后缀为 .d.ts 文件；

// shims-vue.d.ts

declare module '*.vue' {
    import { ComponentOptions } from 'vue';
    const componentOptions: ComponentOptions;
    export default componentOptions;
}

打包友好提示
第一步：安装依赖

npm install friendly-errors-webpack-plugin node-notifier -D

第二步：修改 webpack.config.js 文件

const path = require('path');
+ const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin');
+ const notifier = require('node-notifier');
+ const icon = path.join(__dirname, 'public/icon.jpg');
module.exports = {
  // ...
  plugins: [
    new FriendlyErrorsWebpackPlugin({
      onErrors: (severity, errors) => {
        notifier.notify({
          title: 'webpack 编译失败了~',
          message: `${severity} ${errors[0].name}`,
          subtitle: errors[0].file || '',
          icon,
        });
      },
    }),
    // ...
  ],
};


分析打包文件大小
第一步：安装依赖

npm install webpack-bundle-analyzer -D

第二步：修改 webpack.config.js 文件

+ const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
module.exports = {
  // ...
  plugins: [
    new BundleAnalyzerPlugin(),
    // ...
  ],
};


第三步：修改 package.json 文件

{
    "scripts": {
     // ...
    "analyzer": "webpack --progress"
  },
}

控制台执行 npm run analyzer 系统自动启动打包报告的HTTP服务器；若不想每次都启动，则可以生成 stats.json 文件，后续想查看时在查看。

+ const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
module.exports = {
  // ...
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'disabled',
      generateStatsFile: true
    }),
    // ...
  ],
};

package.json 文件中 scripts 字段新增一条命令：

{
    "scripts": {
     // ...
+    "analyzers": "webpack-bundle-analyzer --port 3000 ./dist/stats.json"
  },
}

通过打包报告可以很直观的知道哪些依赖包大，则可以做做针对性的修改。

打包速度
此插件可以很清晰直观的在控制台显示每个依赖打包所花费的时间

第一步：安装依赖

npm install speed-measure-webpack5-plugin -D

注意：Webpack5 里面配置 speed-measure-webpack-plugin 打包会报错

第二步：修改 webpack.config.js

const SpeedMeasureWebpack5Plugin = require('speed-measure-webpack5-plugin');
const smw = new SpeedMeasureWebpack5Plugin();

module.exports = smw({
	// options
})

缩小打包范围
exclude：排除某些文件，类似黑名单 include：包含哪些文件，类似白名单

如果两者都配置，exclude的优先级比include的高

const path = require('path');

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
          },
        },
        exclude: /node_modules/,
        include: path.resolve(__dirname, 'src')
      },
      // ...
    ],
  },
};


缓存
默认 babel-loader 中可以配置缓存，其他 loader 也想缓存，需下载 cache-loader

第一步：下载依赖

npm install cache-loader -D

第二步：修改 webpack.config.js 文件

const path = require('path');

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
+            cacheDirectory: true
          },
        },
        // ...
      },
      {
        test: /\.css$/,
        use: ['cache-loader', 'style-loader', 'css-loader'],
      }
      // ...
    ],
  },
};

其他
resolve
external
optimization
等等
统一代码规范
统一代码规范包括代码校验、代码格式化、git 提交前校验、编辑器配置等

Eslint
ESLint 是一个开源的 JavaScript 代码检查工具

新增 .eslintrc.js 文件

module.exports = {
  root: true, // 此项是用来告诉eslint找当前配置文件不能往父级查找
  env: {
    node: true, // 此项指定环境的全局变量，下面的配置指定为node环境
  },
  extends: ['plugin:vue/recommended', '@vue/prettier'],
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'vue/no-v-html': 'off',
  },
  parserOptions: {
    parser: 'babel-eslint',
    parser: 'babel-eslint',
    ecmaVersion: 7,
    sourceType: 'module',
    ecmaFeatures: {
      // 添加ES特性支持，使之能够识别ES6语法
      jsx: true,
    },
  },
  overrides: [],
};
  

新增 .eslintignore 文件

# .eslintignore 不需要检查的文件
  
src/assets
src/icons
public
dist
node_modules
  

Perttier
Prettier 是一个代码格式化工具。能够按照我们的规则，将我们的代码格式化

新增 prettier.config.js 文件

module.exports = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  semi: false,
  singleQuote: true,
  quoteProps: 'as-needed',
  jsxSingleQuote: false,
  trailingComma: 'es5',
  bracketSpacing: true,
  jsxBracketSameLine: false,
  arrowParens: 'always',
  htmlWhitespaceSensitivity: 'ignore',
  vueIndentScriptAndStyle: true,
  endOfLine: 'lf',
}

stylelint
stylelint 可以帮助我们规范化 css 的书写，风格统一，减少错误

新增 .stylelintrc.js 文件

module.exports = {
  extends: ['stylelint-config-recess-order', 'stylelint-config-prettier'],
}

EditorConfig
新增 .editorconfig 文件

root = true
  
[*]
charset = utf-8
end_of_line = lf
indent_size = 2
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true
  
[*.md]
trim_trailing_whitespace = false

配置 Git Message
第一步：安装依赖

npm install -g commitizen cz-conventional-changelog
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

安装完毕后，可直接使用 git cz 来取代 git commit

yarn add husky @commitlint/config-conventional @commitlint/cli -D

commitlint : 负责用于对 commit message 进行格式校验
husky : 负责提供更易用的 git hook
第二步：根目录创建 commitlint.config.js

echo 'module.exports = {extends: ["@commitlint/config-conventional"]};' > ./commitlint.config.js

注意：使用 UTF-8 格式, 不然 husky 会报错

第三步：package.json 文件中引入 husky

"husky": {
    "hooks": {
      "commit-msg": "commitlint -e $GIT_PARAMS"
    }
}

第四步：使用

git add .
git cz 选择并输入
git push -u origin branchName即可
自动化发布
let client = require('scp2');
const ora = require('ora');
const chalk = require('chalk');
const spinner = ora(chalk.green('正在发布到服务器...'))

spinner.start()
client.scp('./dist', { // 本地打包的路径
    'host': 'xxx.xxx.x.xxx', // 服务器的IP地址
    'post': '22', // 服务器的IP地址
    'username': 'xxxx', // 用户名
    'password': '*****', // 密码
    'path': '/opt/stu_app_website' // 项目需要部署到服务器的位置
}, err => {
    spinner.stop();
    if(!err) {
        console.log(chalk.green('项目发布完毕'))
    } else {
        console.log('err', err)
    }
})

公司的真实项目是接入了 CI/CD 自动化测试发布
