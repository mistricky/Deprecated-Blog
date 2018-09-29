# Travis CI 持续集成 yarn 执行失败

最近可能没那么多时间来写博客。但是把我最近遇到的一些问题用来写成经验分享也好。

最近使用 Travi CI 来对项目配置持续集成，但是遇到了个问题，因为我项目是用 yarn 来作为包版本管理器，然鹅， 在 Travis 构建中， 出现下面的错误

```
patterns.map is not a function
```

同时

```
The command "eval yarn " failed
```

于是，先是去了 Travis 看了一下官方文档， 是不是对 yarn 有什么特殊的处理。

```
Yarn is supported #
If yarn.lock exists, the default test command will be yarn test instead of npm test.
```

Travis 是支持 yarn 的， 并且在出现 npm test 的时候，会将 npm 替换成 yarn。
soga！

等等！还是没有解决问题啊， 虽然是被支持的， 但是为什么还是会执行 yarn 失败。

下面是 Google 后的结果

```
/me reads the README and upgrades to yarn 1.5.1

...which, of course, fixes the problem...
```

有位同行也遇到了类似的问题， 并且回复到在 yarn 1.5 之后会解决这个问题。

看看 Travis 的执行日志

```
$ yarn --version
1.3.2
```

天呐！`1.3.2`怎么这么低。

现在已经把最开始的 “为什么 yarn 会执行错误” 的问题，变成 “如何升级 yarn” 的问题。嗯，关键点在于，我们将 yarn 升级后还要使 Travis 能够运行我们这个 yarn

分享下我的部分 `.travis.yml`

```
before_install:
  - curl -o- -L https://yarnpkg.com/install.sh | bash -s -- -- version 0.23.2
  - export PATH="$HOME/.yarn/bin:$PATH"
```

`before_install` 会在`install`之前执行， 我们可以在这个钩子上做安装 yarn 的相关工作。

`before_install`的值为一个数组，travis 会数组里的命令依次执行

```
curl -o- -L https://yarnpkg.com/install.sh | bash -s -- -- version 0.23.2	// 使用 curl 安装 yarn
export PATH="$HOME/.yarn/bin:$PATH" // yarn 添加到环境变量
```

好了， 现在已经可以使用比较新的版本了

现在再去 travis 上看的话， 你会发现， 已经绿了！！

祝大家 build:passing !