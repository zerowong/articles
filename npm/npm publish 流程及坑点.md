## 流程

1. 创建账户并在命令行中登录：<https://docs.npmjs.com/creating-a-new-npm-user-account>
2. 验证邮箱
3. 开启两步验证：<https://docs.npmjs.com/configuring-two-factor-authentication>
4. 发布：<https://docs.npmjs.com/creating-and-publishing-unscoped-public-packages>，<https://docs.npmjs.com/creating-and-publishing-scoped-public-packages>

## 坑点

1. 开启两步验证的时候会有一个步骤给出一个二维码，如果没看过文档或者第一次接触两步验证的话会让人很迷惑，其实这里只要下一个验证器 App 就行了  
   ![文档中提到验证器的段落](https://cdn.apasser.xyz/articles/npm%20publish%E6%B5%81%E7%A8%8B%E5%8F%8A%E5%9D%91%E7%82%B9/1.png "文档中提到验证器的段落")  
   这里推荐`Microsoft Authenticator`， IOS 和安卓都有
   ![微软验证器](https://cdn.apasser.xyz/articles/npm%20publish%E6%B5%81%E7%A8%8B%E5%8F%8A%E5%9D%91%E7%82%B9/2.png "微软验证器")
2. 关于`scope`和`unscope`：区别在于带`scope`的包是有`@xxx/`前缀的，例如：`@vue/cli`；

   每个账户会自带一个跟自己名字一样的`scope`，例如账户名为`foo`，则可以发布`@foo/package`形式的包；

   每个组织也是一个`scope`，如果想不以自己的用户名为`scope`发布包，则需要先[创建组织](https://docs.npmjs.com/creating-an-organization)，然后就可以以组织名为`scope`发布包了

3. `package.json`相关配置

   ```json
   // 发布的包中要包含的文件或文件夹
   "files": [
     "esm",
     "lib",
     "umd",
     "README.md"
   ],
   // 包的入口文件
   "main": "lib/index.js",
   // 包的ES6模块入口文件
   "module": "esm/index.js",
   // 发布带scope的包时需要该配置
   "publishConfig": {
     "access": "public"
   },
   ```

4. 403、404 报错。大概就是以下几种情况：没登录、已经有相同名字的包了、包名跟已存在的包过于相似、邮箱未验证、使用了非官方的 npm registry、发布带`scope`的包时没有加`--access public`或者没有上面提到的`publishConfig`配置项、包的`scope`不是自己的用户名或者组织名
