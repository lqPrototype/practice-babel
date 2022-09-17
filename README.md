## 目录：

1. Bable 的重要性
2. Babel 编译流程
3. 如何开发 Babel 插件
4. 四则运算编译器
5. 参考资料
6. 总结

### 1⃣️、Bable 的重要性

1. 打包工具：webpack、rollup、
2. 浏览器兼容性：Babel、
3. 性能：tree-shaking、
4. 类型检查：typescript、
5. 代码规范：eslint
6. 安全： 压缩混淆
7. 友好提示： 代码高亮
8. 文档：docz
9. sourcemap

### 2⃣️、Babel 编译流程

- Babel 简介
- Babel AST 基础知识
- Babel API
- 扩展 Babel Parse
- 扩展 Babel traverse
- 扩展 Bable generate

#### Babel 简介

> 引言： babel 是一个 JS、TS 的编译器，它能把新语法写的代码转换成目标环境支持的语法的代码，并且对目标环境不支持的 api 自动 polyfill。

​

<img src="https://user-images.githubusercontent.com/20496882/52351792-63bef300-2a66-11e9-81d7-905bfe7c4400.png" alt="babel" style="zoom:60%;" />

1. parse：通过**词法分析**和**语法分析**，把源码转成抽象语法树（**AST**）✨
2. transform：遍历 AST，调用各种 transform 插件对 AST 进行增删改 ✨
3. generate：把转换后的 AST 打印成目标代码，并生成 sourcemap

备注：[AST 可视化查看工具](https://astexplorer.net/)

#### Babel AST 基础知识：

> 我们的程序就是由**声明语句**和**语句**或者**表达式**等等构成。

1. 常见的 AST 节点：
   1. Literal： 字面量，const name = '小米' ，分为：StringLiteral、NumericLiteral、BooleanLiteral、RegExpLiteral
   2. Identifier：标识符，变量名、属性名、参数名等
   3. Statement：语句，它是可以独立执行的单位，比如 break、continue、debugger、return，特点：语句末尾一般会加一个分号分隔
   4. Declaration：声明语句是一种特殊的语句，它执行的逻辑是在作用域内声明一个变量、函数、class、import、export 等。
   5. Expression：expression 是表达式，特点是执行完以后有返回值，这是和语句 (statement) 的区别。
   6. es modules：
      1. import： ImportSpicifier、ImportDefaultSpecifier、ImportNamespaceSpcifier。
      2. export： ExportNamedDeclaration、ExportDefaultDeclaration、ExportAllDeclaration。
   7. Program: 最外层程序节点
   8. Directive：指令节点
   9. Comment: 注释节点

疑问：🤔️

1. 为什么 AST 是一棵树 🌲？为什么我们写的代码节点都在 AST 最末位的节点？
2. AST 节点如何自定义？
3. 到底有哪些节点？[@babel/types](https://github.com/babel/babel/blob/main/packages/babel-types/src/ast-types/generated/index.ts)

#### Babel API：

​

- [@babel/parser](https://www.babeljs.cn/docs/babel-parser) : 对源码进行 parse，可以通过 plugins、sourceType 等来指定 parse 语法, 转化为 AST。

- [@babel/traverse](https://www.babeljs.cn/docs/babel-traverse)： 通过 visitor 函数对遍历到的 ast 进行处理，分为 enter 和 exit 两个阶段，具体操作 AST 使用 path 的 api，还可以通过 state 来在遍历过程中传递一些数据

- [@babel/generator](https://www.babeljs.cn/docs/babel-generator): 打印 AST 成目标代码字符串，支持 comments、minified、sourceMaps 等选项。

* [@babel/types](https://www.babeljs.cn/docs/babel-types): 用于创建、判断 AST 节点，提供了 xxx、isXxx、assertXxx 的 api // **重点**
* [@babel/template](https://www.babeljs.cn/docs/babel-template): 用于批量创建节点
* [@babel/code-frame](https://www.babeljs.cn/docs/babel-code-frame)： 打印错误信息，是否高亮、展示啥错误信息。

- [@babel/core](https://www.babeljs.cn/docs/babel-core)：完成整个编译流程，从源码到目标代码，生成 sourcemap。分为同步和异步 API 方法。

​

**<u>备注</u>**： 如果大家对 Babel API 有更深入的了解，可以查看=>Github：**[babel-handbook](https://github.com/jamiebuilds/babel-handbook)**.

#### Babel Parse：

AST 也是有标准的，JS parser 的 AST 大多是 [estree 标准](https://github.com/estree/estree)，从 SpiderMonkey 的 AST 标准扩展而来。babel 的整个编译流程都是围绕 AST 来的，这一节我们来学一下 AST。

熟悉了 AST，也就是知道转译器和 JS 引擎是怎么理解代码的，对深入掌握 Javascript 也有很大的好处。

webpack 、rollup 底层都是基于[acorn](https://github.com/acornjs/acorn)扩展的，如果想看懂 acorn 源码，并且明天它为什么那样写，需要了解编译原理，如果需要自定义语句和 AST 节点。参考如下

- [Definitions（定义）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#definitions%E5%AE%9A%E4%B9%89)
- [Builders（构建器）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-builders)
- [Validators（验证器）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-validators)
- [Converters（变换器）](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-converters)

#### Babel traverse：

```javascript
traverse(AST, {
  Identifier(path, state) {}, // enter
  StringLiteral: {
    enter(path, state) {}, // enter
    exit(path, state) {}, // exit
  },
  "FunctionDeclaration|VariableDeclaration"(path, state) {},
});
```

1. visitor：对象的 value 是对象或者函数(设计模式：**访问者模式**=>当被操作的对象结构比较稳定，而操作对象的逻辑经常变化的时候，通过分离逻辑和对象结构，使得他们能独立扩展)： 1. 如果 value 为函数，那么就相当于是 enter 时调用的函数。 2. 如果 value 为对象，则可以明确指定 enter 或者 exit 时的处理函数。

2. path: path 是遍历过程中的路径，会保留上下文信息，有很多属性和方法.

```javascript
path {
    // 属性：
    node  // 重点
    parent
    parentPath
    scope: { // 重点
      bindings // 重点： 作用域中保存的是声明的变量和对应的值，每一个声明叫做一个binding（绑定）。
      references // 重点
      block: 	| BlockStatement
              | CatchClause
              | DoWhileStatement
              | ForInStatement
              | ForStatement
              | FunctionDeclaration
              | FunctionExpression
              | Program
              | ObjectMethod
              | SwitchStatement
              | WhileStatement
              | ArrowFunctionExpression
              | ClassExpression
              | ClassDeclaration
              | ForOfStatement
              | ClassMethod
              | ClassPrivateMethod
              | StaticBlock
              | TSModuleBlock
      parent // 重点
      parentBlock
      path // 重点

      dump()
      parentBlock()
      getAllBindings()
      getBinding(name)
      hasBinding(name)
      getOwnBinding(name)
      parentHasBinding(name)
      removeBinding(name)
      moveBindingTo(name, scope)
      generateUid(name)
    }
    hub
    container // 重点
    key
    listKey

    // 方法
    get(key)
    set(key, node)
    inList()
    getSibling(key)
    getNextSibling()
    getPrevSibling()
    getAllPrevSiblings()
    getAllNextSiblings()
    isXxx(opts)
    assertXxx(opts)
    find(callback)
    findParent(callback)

    insertBefore(nodes)
    insertAfter(nodes)
    replaceWith(replacement)
    replaceWithMultiple(nodes)
    replaceWithSourceString(replacement)
    remove()

    traverse(visitor, state)
    skip()
    stop()
}
```

2. state：共享数据， AST 节点之间传递数据

```javascript
state {
    file
    opts // 我们写插件传入的参数
}
```

#### Babel generate：

generate 是把 AST 打印成字符串，是一个从根节点递归打印的过程，主要讲下 sourcemap， 首先看下格式：

```
{
　　version : 3,
   file: "out.js",
   sourceRoot : "",
   sources: ["foo.js", "bar.js"],
   names: ["src", "maps", "are", "fun"],
   mappings: "AAgBC,SAAQ,CAAEA"
}
```

重点看 mappping 部分

```javascript
mappings: "AAAAA,BBBBB;;;;CCCCC,DDDDD";
```

每一个分号 `;` 表示一行，多个空行就是多个 `;`，mapping 通过 `,` 分割。

mapping 有五位, 每一位是通过 VLQ 编码：

```
 第一位是目标代码中的列数
 第二位是源码所在的文件名
 第三位是源码对应的行数
 第四位是源码对应的列数
 第五位是源码对应的 names，不一定有
```

参考：https://www.npmjs.com/package/source-map

### 3⃣️、如何开发 Babel 插件：

babel plugin 有两种格式:

    1. 返回对象的函数.
    1. 返回对象。区别： 直接返回一个对象，不用函数包裹，这种方式用于不需要处理参数的情况。

```javascript
// 直接返回对象函数
export default function(api, options, dirname) {
  return {
  	// 继承某个插件
    inherits: parentPlugin,
    // manipulateOptions 用于修改 options，
    manipulateOptions(options, parserOptions) {
        options.lq = '';
    },
    pre(file) {
      this.cache = new Map();
    },
    visitor: {
      StringLiteral(path, state) {
        this.cache.set(path.node.value, 1);
      }
    },
    post(file) {
      console.log(this.cache);
    }
  };
}


// 直接返回对象
export default plugin =  {
    pre(state) {
      this.cache = new Map();
    },
    visitor: {
      StringLiteral(path, state) {
        this.cache.set(path.node.value, 1);
      }
    },
    post(state) {
      console.log(this.cache);
    }
};
```

### 4⃣️、四则运算编译器：

> 遗留：
>
>     1. 为什么AST是一棵树🌲？为什么我们写的代码节点都在AST最末位的节点？
>     1. parse：通过**词法分析**和**语法分析**，把源码转成抽象语法树（**AST**）？

前面说的只是 Babel 大概的一个编译原理，接下来我们通过看下四则运算搞明白这二个问题。

[四则运算语法解析器](https://zhuanlan.zhihu.com/p/112460676)

### 5⃣️、参考资料：

1. 掘金：[掘金-zxg\_神说要有光](https://juejin.cn/user/2788017216685118) <b>主要参考</b>

2. 书籍 📚：[浙江大学-编译原理](https://github.com/QSCTech/zju-icicles/tree/master/%E7%BC%96%E8%AF%91%E5%8E%9F%E7%90%86/%E8%B5%84%E6%96%99)

3. Github：[the-super-tiny-compiler-cn](https://github.com/starkwang/the-super-tiny-compiler-cn)

4. Github：**[babel-handbook](https://github.com/jamiebuilds/babel-handbook)**

### 6、总结：

1. 学习设计模式：访问者模式。
2. babel 插件机制和 preset 机制，都是为了修改 AST 节点。
3. 作用域的灵活运用。
4. 代码如何压缩、高亮、混淆。
5. Acorn 插件机制和扩展强大。
