# 前言

可以方便協同管理不同獨立的庫的生命週期，有更高的複雜性。
在 monorepos 類型的專案中，通常會有多個 JavaScript 代碼庫。
要使用 Babel 轉譯這些代碼庫，我們需要對 Babel 進行配置
。不同包的管理工具(npm workspace)，都提供了 workspace 功能。

### pnpm 有效避免幽靈依賴
安裝得比較快
處理不同依賴時，是以link形式


#### pnpm 有效避免幽靈依賴 
幽靈依賴:(yarn 跟 npm 歷史遺留問題)
沒有在依賴中顯示聲明，套件 A 可能因為其他套件 B 的依賴而間接獲取套件 C，
儘管套件 A 並沒有在它的 package.json 中明確聲明套件 C 為其依賴。
這會導致未來的某次依賴更新或重新安裝時，因為套件 C 的版本變動或不存在而出現問題。

範例：
假設有以下依賴關係：

packageA 依賴 packageB
packageB 依賴 packageC


### npm 和 yarn 的扁平化依賴管理方式
在扁平化依賴管理下， packageC 可能被安裝在根目錄的 node_modules 中，
而不是嵌套在 packageB 的 node_modules 中

```
project-root/
├── node_modules/
│   ├── packageA/       # 提升到頂層
│   ├── packageB/
│   └── packageC/
```

這樣 packageA 也能夠訪問到 packageC，即使它並沒有在 package.json 中聲明 packageC 為依賴。
這樣的使用方式是危險的，因為如果將來 packageB 移除或更新了它對 packageC 的依賴， 
packageA 也會因找不到 packageC 而發生錯誤。

```
// packageA 的文件中
const functionC = require('packageC'); // 沒有在 packageA 的 package.json 中聲明 packageC
```



