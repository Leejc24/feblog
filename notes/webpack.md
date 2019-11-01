### webpack4.x相关
+ webpack.config.js配置项简介
    - Entry：入口文件配置，Webpack 执行构建的第一步将从 Entry 开始，完成整个工程的打包。
    - Module：模块，在Webpack里一切皆模块，Webpack会从配置的Entry开始递归找出所有依赖的模块，最常用的是rules配置项，功能是匹配对应的后缀，从而针对代码文件完成格式转换和压缩合并等指定的操作。
    - Loader：模块转换器，用于把模块原内容按照需求转换成新内容，这个是配合Module模块中的rules中的配置项来使用。
    - Plugins：扩展插件，在Webpack构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。(插件API)
    - Output：输出结果，在Webpack经过一系列处理并得出最终想要的代码后输出结果，配置项用于指定输出文件夹，默认是./dist。
    - DevServer：用于配置开发过程中使用的本机服务器配置，属于webpack-dev-server这个插件的配置项。

    - devtool:打包后的代码和原始代码存在较大的差异，此选项控制是否生成以及如何生成sourcemap
    - devserver：通过配置devserver选项，可以开启一个本地服务器
    - watch：启用watch模式后，webpack将持续监听热河已经解析文件的更改，开发是开启会很方便
    - watchoption：用来定制watch模式的选项
    - performance：打包后命令行如何展示性能提示，如果超过某个大小是警告还是报错

+ webpack打包流程简介
    - 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
    - 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
    - 确定入口：根据配置中的 entry 找出所有的入口文件
    - 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
    - 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
    - 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
    - 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。
    - 在以上过程中，webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 webpack 提供的 API 改变 webpack 的运行结果。

+ webpack 性能优化
    - 动态链接库DLL
    - happypack 多线程打包（适用于大的项目，文件较小时可能会变慢）
    - 多页面打包提取公共代码
    - 优化 Loader
    - 优化Loader的文件搜索范围
    - 把Babel编译过得文件缓存起来，下次只需要编译更好过得文件即可
    - webpack 自带优化
        - three shaking 仅支持import语法在生产模式下会自动去除掉没有用的代码
        - scop hosting 作用域提升、简化代码

基本的webpack4.x 常用配置文件 webpack.config.js
```
const path = require('path');
const CopyWebpackPlugin = require('copy-webpack-plugin') // 复制静态资源的插件
const CleanWebpackPlugin = require('clean-webpack-plugin') // 清空打包目录的插件
const HtmlWebpackPlugin = require('html-webpack-plugin') // 生成html的插件
const ExtractTextWebapckPlugin = require('extract-text-webpack-plugin') //CSS文件单独提取出来
const webpack = require('webpack')

module.exports = {
    entry: {
        index: path.resolve(__dirname, 'src', 'index.js'),
        page: path.resolve(__dirname, 'src', 'page.js'),
        vendor:'lodash' // 多个页面所需的公共库文件，防止重复打包带入
    },
    output:{
        publicPath: '/',  //这里要放的是静态资源CDN的地址
        path: path.resolve(__dirname,'dist'),
        filename:'[name].[hash].js'
    },
    resolve:{
        extensions: [".js",".css",".json"],
        alias: {} //配置别名可以加快webpack查找模块的速度
    },
    module: {
        // 多个loader是有顺序要求的，从右往左写，因为转换的时候是从右往左转换的
        rules:[
            {
                test: /\.css$/,
                use: ExtractTextWebapckPlugin.extract({
                    fallback: 'style-loader',
                    use: ['css-loader', 'postcss-loader'] // 不再需要style-loader放到html文件内
                }),
                include: path.join(__dirname, 'src'), //限制范围，提高打包速度
                exclude: /node_modules/
            },
            {
                test:/\.less$/,
                use: ExtractTextWebapckPlugin.extract({
                    fallback: 'style-loader',
                    use: ['css-loader', 'postcss-loader', 'less-loader']
                }),
                include: path.join(__dirname, 'src'),
                exclude: /node_modules/
            },
            {
                test:/\.scss$/,
                use: ExtractTextWebapckPlugin.extract({
                    fallback: 'style-loader',
                    use:['css-loader', 'postcss-loader', 'sass-loader']
                }),
                include: path.join(__dirname, 'src'),
                exclude: /node_modules/
            },
            {
                test: /\.jsx?$/,
                use: {
                    loader: 'babel-loader',
                    query: { //同时可以把babel配置写到根目录下的.babelrc中
                      presets: ['env', 'stage-0'] // env转换es6 stage-0转es7
                    }
                }
            },
            { //file-loader 解决css等文件中引入图片路径的问题
            // url-loader 当图片较小的时候会把图片BASE64编码，大于limit参数的时候还是使用file-loader 进行拷贝
                test: /\.(png|jpg|jpeg|gif|svg)/,
                use: {
                  loader: 'url-loader',
                  options: {
                    outputPath: 'images/', // 图片输出的路径
                    limit: 1 * 1024
                  }
                }
            }
        ]
    },
    plugins: [
        // 多入口的html文件用chunks这个参数来区分
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname,'src','index.html'),
            filename:'index.html',
            chunks:['index', 'vendor'],
            hash:true,//防止缓存
            minify:{
                removeAttributeQuotes:true//压缩 去掉引号
            }
        }),
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname,'src','page.html'),
            filename:'page.html',
            chunks:['page', 'vendor'],
            hash:true,//防止缓存
            minify:{
                removeAttributeQuotes:true//压缩 去掉引号
            }
        }),
        new webpack.ProvidePlugin({
            _:'lodash' //所有页面都会引入 _ 这个变量，不用再import引入
        }),
        new ExtractTextWebapckPlugin('css/[name].[hash].css'), // 其实这个特性只用于打包生产环境，测试环境这样设置会影响HMR
        new CopyWebpackPlugin([
            {
                from: path.resolve(__dirname, 'static'),
                to: path.resolve(__dirname, 'dist/static'),
                ignore: ['.*']
            }
        ]),
        new CleanWebpackPlugin([path.join(__dirname, 'dist')]),
    ],
    devtool: 'eval-source-map', // 指定加source-map的方式
    devServer: {
        contentBase: path.join(__dirname, "dist"), //静态文件根目录
        port: 3824, // 端口
        host: 'localhost',
        overlay: true,
        compress: false // 服务器返回浏览器的时候是否启动gzip压缩
    },
    watch: true, // 开启监听文件更改，自动刷新
    watchOptions: {
        ignored: /node_modules/, //忽略不用监听变更的目录
        aggregateTimeout: 500, //防止重复保存频繁重新编译,500毫米内重复保存不打包
        poll:1000 //每秒询问的文件变更的次数
    },
}
```