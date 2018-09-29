# 【使用Nest.js开发】基于TypeScript的高性能框架

最近接手一个后台管理的小项目，需求就是做一个网站的后台管理系统。想来想去，用java吧，太重，用node好了，于是想起前几天接触的nest框架，加上本人对typescript也是比较喜欢的，于是着手开始使用nest开发。

Nest是构建高效，可扩展的[Node.js](http://nodejs.org/)服务器端应用程序的框架。它使用现代JavaScript，使用[TypeScript](http://www.typescriptlang.org/)（保留与纯JavaScript的兼容性）构建， 并结合了OOP（面向对象编程），FP（功能编程）和FRP（功能反应编程）的元素。

在引擎盖下，Nest使用[Express](https://expressjs.com/)，可以轻松使用可用的无数第三方插件。

需要注意的是nest是基于Express之上的。

### first

构建一个nest应用之前，安装其所需要的依赖

```
$ npm i --save @nestjs/core @nestjs/common @nestjs/microservices @nestjs/websockets @nestjs/testing reflect-metadata rxjs
```

里面包含了nest的核心模块，common和core，微服务模块micro services ，websocket模块，测试模块testing，以及typescript提供用来反射元数据的reflect-metadata，还有基于响应式数据流的rxjs

由于nest是基于typescript的，因此，我们需要一个tsconfig.json，来描述ts的配置。

```
tsc --init
```

```
{
  "compilerOptions": {
    /* Basic Options */
    "target": "es6",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'. */
    "module": "commonjs",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "lib": ["es2015"],                             /* Specify library files to be included in the compilation:  */
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    // "jsx": "preserve",                     /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
    // "declaration": true,                   /* Generates corresponding '.d.ts' file. */
    // "sourceMap": true,                     /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist",                        /* Redirect output structure to the directory. */
    "rootDir": "./src",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "removeComments": true,                /* Do not emit comments to output. */
    // "noEmit": true,                        /* Do not emit outputs. */
    // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,                            /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                 /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,              /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,                /* Report errors on unused locals. */
    // "noUnusedParameters": true,            /* Report errors on unused parameters. */
    // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                       /* List of folders to include type definitions from. */
    // "types": [],                           /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */

    /* Source Map Options */
    // "sourceRoot": "./",                    /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "./",                       /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
    "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */
  }
}
```

需要注意的是，这里的target最好给出es6，这是因为nest的middleware需要es6的支持

创建一个src作为源文件夹，dist为编译后的版本

为了能更方便的编译运行nest，创建一个index.js作为入口文件

**index.js**

```
require('ts-node/register')
require('./src/main')
```

**ps:main为该项目的入口文件**

### 正式开始

准备工作都做完了，现在来正式开始我们的第一个nest应用。

上一步在index.js里require了main文件，我们现在正式来创建一个main
main为nest应用的入口文件，作为启动nest应用的入口，需要一个入口模块作为启动依据

首先创建我们的入口模块

**src/app.module.ts**

```
import { Module } from "@nestjs/common";

@Module({
    
})
export class ApplicationModule{ }
```

当前还什么都没有，等会儿我们会向模块里导入一个控制器

然后创建main.ts

**src/main.ts**

```
import { NestFactory } from '@nestjs/core';
import { ApplicationModule } from './app.module';

async function bootstrap(){
    const app = await NestFactory.create(ApplicationModule)
    app.listen(3000)
}

bootstrap();
```

创建一个bootstrap函数来启动我们的应用，启动NestFactory用于把applicationModule实例化，并且创建一个http服务器，然后监听它的3000端口，不要忘了，nest是基于express的。

运行一下看下效果：

```
node index
```

我们的nest应用已经能够运行起来了，但是目前里面没有任何东西

### Controller

控制器用于描述业务逻辑，客户端通过web api访问服务器，服务器根据访问的路由把请求分发至响应的控制器，然后控制器，最后封装成数据模型返回给客户端，典型的MVC不是吗

在src目录下创建一个user文件夹，然后创建一个user.controller.ts

```
$ mkdir user
$ touch user.controller.ts
```

**src/user/user.controller.ts**

```
import { Controller, Get, Post } from "@nestjs/common";

@Controller('user')
export class UserController{
    @Get('all')
    findAll(){
        return ['all']
    }

    @Post('add')
    addUser(){
        return ['add new user'];
    }
}
```

Controller装饰器描述一个路由前缀
比如get装饰器里传入一个all，那么当我们访问**/user/all**的时候，请求就会被分发到findAll里进行处理。
这样我们能更方便的提供良好的RESTful API

现在Controller 还不能被应用到nest应用中，我们希望controller仅仅是处理路由，而复杂的业务逻辑交给service来做，nest里提供了和angular相似的依赖注入，我们可以将任何以Component装饰器装饰的类注入到我们需要的地方

创建一个user.service.ts来处理controller里的业务逻辑

**src/user.service.ts**

```
import { Component } from "@nestjs/common";

@Component()
export class UserService{
    findAll():Array<string>{
        //find all logic TODO
        return ['find all']
    }
    
    addUser():Array<string>{
        return ['add a new user']
    }
}
```

将user.service.ts注入到user.controller.ts中

```
import { Controller, Get, Post } from "@nestjs/common";
import { UserService } from "./user.service.ts"

@Controller('user')
export class UserController{
	
	//inject user service
	controller(
		private user:UserService
	){
        
	}
	
    @Get('all')
    findAll():Array<string>{
        return this.user.findAll();
    }

    @Post('add')
    addUser():Array<string>{
        return this.user.addUser();
    }
}
```

现在再创建一个user.module.ts，将它们导入到同一模块下

**src/user/user.module.ts**

```
import { Module } from "@nestjs/common";
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
    components:[
        UserService
    ],
    controllers:[
        UserController
    ]
})
export class UserModule{ }
```

再将user.module导入到主模块中

```
import { Module } from "@nestjs/common";
import { UserModule } from './user/user.module'

@Module({
    imports:[
        UserModule
    ]
})
export class ApplicationModule{ }
```

现在来测试我们的第一个nest应用

```
$node index
[Nest] 21906   - 2018-3-25 15:03:39   [NestFactory] Starting Nest application...
[Nest] 21906   - 2018-3-25 15:03:39   [InstanceLoader] ApplicationModule dependencies initialized +6ms
[Nest] 21906   - 2018-3-25 15:03:39   [InstanceLoader] UserModule dependencies initialized +1ms
[Nest] 21906   - 2018-3-25 15:03:39   [RoutesResolver] UserController {/user}: +21ms
[Nest] 21906   - 2018-3-25 15:03:39   [RouterExplorer] Mapped {/all, GET} route +2ms
[Nest] 21906   - 2018-3-25 15:03:39   [RouterExplorer] Mapped {/add, POST} route +1ms
[Nest] 21906   - 2018-3-25 15:03:39   [NestApplication] Nest application successfully started +0ms
```

成功启动了。通过浏览器访问**http://localhost:3000/user/all**，发现返回["find all"]

至此，我们的第一个nest应用大功告成。

### 最后

小生文笔略糙，希望能够帮到正在学习nest的同行