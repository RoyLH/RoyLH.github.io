---
title: 前端自动化---Grunt
tags:
  - 前端自动化
  - 前端工程化
  - 持续集成
categories:
  - 前端自动化
  - 前端工程化
  - 持续集成
abbrlink: 860
date: 2019-06-03 22:54:08
---
![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1559583956137&di=0b4cfc04029c79f70d95157839f24e44&imgtype=0&src=http%3A%2F%2Fseanamarasinghe.com%2Fwp-content%2Fuploads%2F2014%2F03%2Fgrunt-1050x360.jpg)

## 你需要准备的环境

---
- node.js
---

## (一) 安装

---

```
npm install grunt --save

```

---

## (二) 配置gruntfile.js

---

需要修改两处：

```

module.exports = () => {
  //配置任务 所有插件的配置信息
  grunt.initConfig({
    // 获取 package.json的信息
    pkg:grunt.file.readJSON(package,json);
  });
  // 告诉grunt 当我们在终端输入grunt时 它需要做些什么
  grunt.registerTask('default',[]);
}；

```

---

## (三) grunt相关插件的介绍

---

grunt官网的插件列表页面 http://www.gruntjs.net/plugins
插件很多。。。
分为两大类
第一类：
grunt团队贡献  有“contrib-”前缀 在插件列表中带有*号
第二类
第三方提供的插件
常用的 排名靠前的：
contrib-jshint           JavaScript语法错误检查
contrib-watch          实时监控文件的变化 调用相应的任务重新执行
contrib-clean           清空文件 文件夹
contrib-uglify		压缩JavaScript代码
contrib-copy		复制文件 文件夹
contrib-concat    	合并多个文件的代码到一个文件中
karma 			前端自动化测试工具

---

## (四) 使用uglify插件（压缩JavaScript代码）

---
- 安装
```
npm insatll grunt-contrib-uglify --save-dev
```
- 任务： 压缩js文件
  - step1：在grunt.initConfig方法中配置uglify的配置参数
    ```
      module.exports = () => {
        //配置任务 所有插件的配置信息
        grunt.initConfig({
          // 获取 package.json的信息
          pkg:grunt.file.readJSON(package,json);

          // uglify插件的配置信息
          uglify: {
            options: {
              stripBanners: true,
              banner: '/*! <%=pkg.name>-<%=pkg.version%>.js <%=grunt.template.tody("yyyy-mm-dd")%>*/\n'
            },
            // options规定允许生成的压缩文件带banner，即在生成压缩文件第一行加一句话说明。注意，其中使用到了pkg获取package.json的内容。
            build: {
              src: 'src/test.js',
              dest: 'build/<%=pkg.name%>-<%pkg.version%>.js.min.js
            }
            // build配置源文件和目标文件。src规定了要压缩谁 desc规定了压缩之后会生成谁 注意，这里将目标文件的文件名通过pkg的name和version来命名。
          }
        });
        // 告诉grunt 当我们在终端输入grunt时 它需要做些什么
        grunt.registerTask('default',[]);
      }；
    ```
  - step2:在grunt.initConfig方法之后，要让grunt去加载这一个插件。
    ```
      grunt.initConfig({
        ...
      });

      // 告诉grunt我们将使用插件
      grunt.loadNpmTasks('grunt-contrib-uglify');
    ```
  - step3: 在grunt命令执行时，要不要立即执行uglify插件？如果要，就写上，否则不写。
    ```
      grunt.initConfig({
        ...
      });

      // 告诉grunt我们将使用插件
      grunt.loadNpmTasks('grunt-contrib-uglify');
      // 告诉grunt当我们在终端输入grunt时需要做些什么（注意先后次序）
      grunt.registerTask('default',['uglify']);
    ```





  完成
---

## (五) 使用jshint插件（检查JavaScript语法错误）

---

- 安装
```
npm insatll grunt-contrib-jshint --save-dev
```
- 任务：检查当前test.js的语法错误
  - step1:在grunt.initConfig方法中配置jshint的配置参数
    ```
      module.exports = () => {
        //配置任务 所有插件的配置信息
        grunt.initConfig({
          // 获取 package.json的信息
          pkg:grunt.file.readJSON(package,json);

          // uglify插件的配置信息
          uglify: {
            ...
          },

          // jshint插件的配置信息
          jshint: {
            build: ['Gruntfile.js', 'src/*.js'],
            // build描述了jshint要检查哪些js文档的语法
            options: {
              jshint: '.jshintrc'
            }
            // options描述了要通过怎么的规则检查语法
            // 这些规则的描述文件就保存在网站根目录下的一个叫做“.jshintrc”的文件中
            // 在网站的根目录下面添加上这个文档，并且填写上文件内容。
          }
        });
        // 告诉grunt 当我们在终端输入grunt时 它需要做些什么
        grunt.registerTask('default',[]);
      }；
    ```
  - step2:在 grunt.initConfig 方法之后，要让grunt去加载这一个插件。注意没有先后顺序
    ```
    // 告诉grunt我们将使用插件
      grunt.loadNpmTasks('grunt-contrib-uglify');
      grunt.loadNpmTasks('grunt-contrib-jshint');
    ```
  - step3：配置grunt命令启动时，要执行的任务，这里注意先后顺序。是希望先检查语法呢？还是先合并呢？——应该先检查语法比较好，因为语法不对，合并了有什么意义
    ```
      grunt.registerTask('default',['jshint', uglify']);
    ```

  完成
---

## (六) 使用csslint插件（检查css语法错误）

---

- 安装
```
npm insatll grunt-contrib-csslint --save-dev
// 只不过csslint依赖于一个叫做“.csslintrc”的文件作为语法检验的规则。
```

---