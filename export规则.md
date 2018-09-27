- import 校验
  - import 路径中 (排除 `@` 开头的模块, 如 `@angular/core`), 以 `@` 开头的文件夹前不应有，除 `./`, `../` 外的其他具名文件夹. 如 `./foo/@bar`.
- export 校验
  - export 路径中不应包含以 `@` 开头的路径.
  - index.js, index.ts 中, 验证同目录下所有不以 `@` 开头的项都以 `export * from './xxx';` 的形式被导出.
    - 需要导出的项包括: `xxx.js` 或 `xxx.ts`, 以及包含 `index.js` 或 `index.ts` 的文件夹.
    - 错误 1: 多导出了 `@` 开头的文件/文件夹. 提供 fixer 移除 (这里可能要考虑提供一个 replacement 数组作为 fixer 参数).
- - - 错误 2: 少导出了需要导出的项. 提供 fixer 添加.

      

