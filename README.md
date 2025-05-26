# 前言

1. 多個 React 應用或元件庫可以放在同一個倉庫，直接引用彼此的程式碼，無需經過發佈 npm 套件等繁瑣流程，修改共用元件後所有依賴專案可同步受益。

2. 依賴與版本一致性，所有子專案（如多個 React 應用、元件庫）都能使用同一份 React 及相關依賴，避免「A 用 React 18、B 用 React 17」導致的衝突與難以維護。

3. 統一開發流程與工具設定，測試、打包、Lint、CI/CD 等工具與設定可以集中管理，減少重複設定與維護成本，提升團隊協作效率。

4. 更容易的重構與透明度，程式碼集中於一處，跨專案重構、修 bug、追蹤依賴關係更直接、透明。

5. 團隊協作與溝通提升，多個團隊在同一倉庫協作，程式碼風格、流程容易統一，溝通成本降低。

6. 建構與部署效率提升，可以利用 monorepo 工具（如 Lerna、Nx、Turborepo）只針對變更過的部分建構與部署，提升效能。

補充說明
Monorepo 也有其缺點，例如倉庫規模大時 Git 操作效能下降、權限細緻度較難控制等。


### pnpm 優點
1. 安裝得比較快
2. 處理不同依賴時，是以link形式
3. 沒有幽靈依賴:(yarn 跟 npm 歷史遺留問題)

### 幽靈依賴 說明

沒有在依賴中顯示聲明，套件 A 可能因為其他套件 B 的依賴而間接獲取套件 C，
儘管套件 A 並沒有在它的 package.json 中明確聲明套件 C 為其依賴。
這會導致未來的某次依賴更新或重新安裝時，因為套件 C 的版本變動或不存在而出現問題。

```
// 範例：
// 假設有以下依賴關係：
packageA 依賴 packageB
packageB 依賴 packageC
```

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



