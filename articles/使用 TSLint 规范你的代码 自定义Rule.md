# 使用 TSLint 规范你的代码 自定义Rule

TSLint可以帮你规范你的代码，配合 vs code 上的 TSLint 插件，可以帮你治疗你的代码洁癖。

为了统一公司内的代码风格，最近公司也要开发一些 Rule，配合 Prettier 还不是美滋滋？而我就是负责这个 Rule 的开发，TSlint 在国内的资料少之又少， 并且 TSLint 自己的文档对开发自定义 Rule 也是描述的很模糊， 于是避免不了各种找资料，看源码。当你会写第一个 Rule 的时候， 后面的开发就比较顺畅了。现在也还是正在开发，于是抽空写一篇博文出来，帮助也想写自定义 Rule 的同志们，少走一些弯路吧， 有兴趣的话，也可以移步我们正在开发的 [MagicSpace](https://github.com/makeflow/magicspace)，虽然还在开发中，Star✨一下呗 :)

## 从一个官方的例子看起

进入正题。

由于官方的例子也说的比较明了，先看看官方给出的例子

```
import * as ts from "typescript";
import * as Lint from "tslint";

export class Rule extends Lint.Rules.AbstractRule {
    public static FAILURE_STRING = "import statement forbidden";

    public apply(sourceFile: ts.SourceFile): Lint.RuleFailure[] {
        return this.applyWithWalker(new NoImportsWalker(sourceFile, this.getOptions()));
    }
}

// The walker takes care of all the work.
class NoImportsWalker extends Lint.RuleWalker {
    public visitImportDeclaration(node: ts.ImportDeclaration) {
        // create a failure at the current position
        this.addFailure(this.createFailure(node.getStart(), node.getWidth(), Rule.FAILURE_STRING));

        // call the base version of this visitor to actually parse this node
        super.visitImportDeclaration(node);
    }
}
```

假设你要写的 Rule 叫 noImportRule.ts, 在该文件中， 你必须要导出名称为Rule的类，并且显示的继承 Rules.AbstractRule， 然后实现它的 apply方法，apply方法有一个sourceFile的参数，该参数是当前正应用该Rule的源文件。返回值为一个RuleFailure，在TSLint中，不符合 Rule 的被视为一个 Failure ，这个返回值就是一个Failure数组，当然， 我们并不直接返回它，而是返回一个Walker 。

Walker是一个Rule中的核心部分，是由它来判断哪里不符合规定，该在哪里加failure 之类的工作。声明一个 NoImportsWalker，这里写的是继承RuleWalker，RuleWalker相当于一个访问者，你可以通过它，访问到你想要的语句，比如 visitImportDeclaration 就是访问到该源文件的所有import声明语句。参数 node 拿到的就是import的表达式，node代表一个AST的节点（在TSLint中，检查规则之前，首先会把你的源代码转换为AST，然后通过一些方法来提取AST中的节点， 在 TSLint 中用的是 TSUtil 这个工具，TSLint 这么火的同时，TSUtil 才只有几十个star， 看来是TSLint没有带火TSUtil啊，可能是大家都如何提取AST中的节点不感兴趣，比较感兴趣的还是TSLint 吧），之后拿到这个节点就能执行一些这个节点上的方法，比如说，用getText() 获取这个这点的内容，或者是用getChildren，getParent 获取这个节点的父子节点等，相似的方法还有很多。

代码中的` this.addFailure(this.createFailure(node.getStart(), node.getWidth(), Rule.FAILURE_STRING));`就是调用addFailure 达到一个添加错误的目的。

这个rule的功能就是禁止import 语句，运行看看效果？

比如我这里随便导入个模块

```
import * as Foo from './foo'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~				[import statement forbidden]
```

可以看到这个语句已经被 TSLint 报错了， 这是 addFailure 的作用

## Walker 和 Function

一个简单的Rule就写好了。这样看是不是很简单？

其实一般在做自定义 Rule 的时候，我们一般不会直接继承 RuleWalker， 而是会继承 AbstractWalker，然后实现它的 walk 方法，比如下面是我[MagicSpace](https://github.com/makeflow/magicspace)里的一个代码片段

```
class ExplicitReturnTypeWalker extends AbstractWalker<undefined> {
  /** 装载 */
  walk(sourceFile: TypeScript.SourceFile): void {
    let cb = (node: TypeScript.Node): void => {
      if (
        (TypeScript.isFunctionDeclaration(node) ||
          TypeScript.isArrowFunction(node) ||
          TypeScript.isFunctionExpression(node) ||
          TypeScript.isMethodDeclaration(node)) &&
        node.type
      ) {
      }
      TypeScript.forEachChild(node, cb);
    };

    TypeScript.forEachChild(sourceFile, cb);
  }
}
```

这个 walker是用来获取到一些我想要的函数节点，比如函数表达式，箭头函数，函数声明语句，方法声明。TSLint 作规则检查的时候就会调用walk，因此，你可以在里面写一些判断逻辑，以达成你的规则判定要求。

除此之外，如果你的 rule 比较简单， 你大可不必去创建一个walker类，而是可以直接用applyWithFunction，功能上和walker差不多，只是用一个函数来替代walker。

```
protected applyWithFunction(sourceFile: ts.SourceFile, walkFn: (ctx: WalkContext<void>) => void): RuleFailure[];
    protected applyWithFunction<T>(sourceFile: ts.SourceFile, walkFn: (ctx: WalkContext<T>) => void, options: NoInfer<T>): RuleFailure[];
    protected applyWithFunction<T, U>(sourceFile: ts.SourceFile, walkFn: (ctx: WalkContext<T>, programOrChecker: U) => void, options: NoInfer<T>, checker: NoInfer<U>): RuleFailure[];
```

比如

```
applyWithFunction(souceFile, ctx => {
    // some logic
})
```

这个函数就相当于 Walker 里面的walk方法，在检查规则的时候也会去调用这个函数，ctx参数就相当于是Walker的上下文。如果要调用 addFailure 的话，不是通过`this.addFailure`去调用，而是通过`ctx.addFailure`去进行调用。

基本可以实现和walker一样的功能。

相对逻辑比较简单的 Rule， 用`applyWithFunction` 可能更加轻便。

## Metadata

除此之外，我们还可以在Rule的类中写元数据，用于来描述这个 Rule。看一个我写的一个 Metadata

```
static metadata: IRuleMetadata = {
    ruleName: 'import-groups',
    description: 'Validate that module imports are grouped as expected.',
    optionsDescription: '',
    options: {
      properties: {
        groups: {
          items: {
            properties: {
              name: {
                type: 'string',
              },
              test: {
                type: 'string',
              },
            },
            type: 'object',
          },
          type: 'array',
        },
        ordered: {
          type: 'boolean',
        },
      },
      type: 'object',
    },
    optionExamples: [
      [
        true,
        {
          groups: [
            {name: 'node-core', test: '$node-core'},
            {name: 'node-modules', test: '$node-modules'},
          ],
          ordered: true,
        },
      ],
    ],
    type: 'maintainability',
    hasFix: true,
    typescriptOnly: false,
  };
```

这里的字段的意思还是大家下来看看文档，这里就不再赘述了，有很多字段是可选的，看自己的情况需要来选。具体的可以看`rule.d.ts`中的IMetadata接口

## 查找对应的表达式

有的时候，我们需要查找对应的表达式，比如我们想收集整个文件里的 import 声明语句，可以用 kind 来作判断。kind 是什么？

`Node.kind` 是一个表示这个节点语义类型的整数，枚举类 `ts.SyntaxKind` 有完整的对应关系。

Oh，我们来看一个实际的例子。

```
import * as TypeScript form 'typescript'

for (let statement of sourceFile.statements) {
      if (statement.kind === Typescript.SyntaxKind.ImportDeclaration) {
        //...
      }
```

这里可以通过对比每个 statement 的 kind 来获取所有的 import 的声明语句，不过在 tsutil 里封装了一些可以直接调用的方法，比如`isImportDeclaration`

那么上面的代码就可以改成

```
import * as TypeScript form 'typescript'

for (let statement of sourceFile.statements) {
      if (isImportDeclaration(statement)) {
        //...
      }
```

还有 `findImports`方法可以帮助我们快速的找到所有的 import 语句

```
for (const expression of findImports(
      sourceFile,
      ImportKind.AllStaticImports,
    )) {
      //...
    }
```

是不是方便很多，当然，不止 import 

## 总结

TSLint 当然是 TS 必备的一个风格检查的工具， 但是 TSLint 自带的一些规则肯定不能满足我们的有些奇怪的要求，那么可能就需要自己写一个 TSLint Rule，帮助我们在风格层面来规范代码。暂时只想到要写这些，国内的 TSLint 资料太少了，这个博文只是帮助大家找到一点思路， 当然，还是得结合文档和源码来构建自己的 TSLint 规则。