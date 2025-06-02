# 專案當前進度 (最近一次更新:2025/05/27)

- 已配置eslint 跟 prettier 所需
- 已測試過可以正常檢查 代碼風格跟規範
- 預計下次更新=> 處理 husky

## 前言

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

### 當時配置

```
npm install -g pnpm // 先確定有安裝 pnpm
pnpm -v // 10.11.0
pnpm init // 生成package.json
```

### Eslint 代碼規範

```
// -d 是 --save-dev ,
// --workspace 的縮寫，這個參數用在 npm 支援的 monorepo 專案（例如包含多個子專案的 workspace 結構），
// 指定要在哪個 workspace 裡安裝這個套件。如果你在 monorepo 根目錄下執行並加上 -w，可以指定要安裝到哪個子專案；
// 如果沒指定，預設就是當前目錄
pnpm i eslint -d -w
npx eslint --init // eslint.config.mjs
<!-- Flat Config 支援多種副檔名
eslint.config.mjs，是因為 ESLint v9 以後預設使用 Flat Config
根據官方文件，Flat Config 可以是 eslint.config.js、eslint.config.mjs、eslint.config.cjs 等。eslint.config.mjs 就是這種新格式的其中一種 -->

<!-- 新舊格式的差異
舊的 .eslintrc.json 屬於物件型設定，新的 Flat Config 則是以 JavaScript 模組方式匯出設定陣列，彈性更高，也更容易整合現代化的 Node.js 專案。 -->
```

#### npx eslint --init 會報錯

1. 因為之前 pnpm i eslint 的時候後面有加 -w，所以接下來的套件安裝也都要加

```
// ESLint 对 TypeScript 的解析，使用了 @typescript-eslint/parser
// parser 字段指定让 ESLint 使用自定义的解析器 @typescript-eslint/parser


pnpm i -d -w @typescript-eslint/eslint-plugin@latest, @typescript-eslint/parser@latest

```

#### 加了 -w 之後 @typescript-eslint/eslint-plugin@latest 仍然會下載失敗

參考 C-1 錯誤描述
原因: @latest 這個寫法是 pnpm 不能解析的，刪除掉就可以了

```
<!-- 安裝ts-eslint插建 -->
pnpm i -d -w @typescript-eslint/eslint-plugin
```

#### 如果出現 missing peer typescript@'\*'

代表很多庫依賴了其他庫，但這些其他庫，又沒必要安裝。
既需要依賴，又不需要安裝，就把它也裝起來就好了。

```
pnpm i -d -w typescript
```

### prettier 代碼風格

```
pnpm i prettier -d -w
```

#### 新建.prettierrc.json 配置文件

```
{
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": true,
  "singleQuote": true,
  "semi": true, // 最後一行加分號
  "trailingComma": "none",
  "bracketSpacing": true
}

```

#### 由於 eslint也可以做風格檢查，所以可能跟prettier會產生衝突

所以需要將 prettier 集成到 eslint 中

```
// https://juejin.cn/post/7239987776552714300
pnpm i eslint-config-prettier eslint-plugin-prettier -d -w
```

#### package.json

```
--quiet // 不輸出反饋的形式
```

#### 到這個階段，會發現還是有缺少一些套件

1. globals
2. typescript-eslint
   這兩個都是接下來 `pnpm run lint` 時會產生的套件缺失報錯，記得都按照 `pnpm i xxx -d -w` 方式補下載即可

#### 錯誤訊息查詢

- [C-1]ERR_PNPM_SPEC_NOT_SUPPORTED_BY_ANY_RESOLVER  @typescript-eslint/eslint-plugin@latest, isn't supported by any available resolver.

#### 安装 husky

```
pnpm add --save-dev husky
```

#### 初始化 Husky

```
pnpm exec husky init
```

#### 想讓它在 commit 前執行 pnpm lint

.husky\pre-commit 直接在路徑中覆script命令，之後每次執行 git commit 前，Husky 就會自動執行 pnpm lint，效果等同 id .husky/pre-commit "pnpm lint"，這樣確保了每次commit前都是在規範下才上傳至codebase。做到了即時檢查的效果。

```
pnpm lint
```

#### 對commit規範訊息進行檢查

只要安裝 @commitlint/cli 和 @commitlint/config-conventional 就能正常運作，不需要再安裝 commitlint
.commitlintrc.js 和 commitlint.config.js 其實都是 commitlint 支援的設定檔案格式，兩者都可以用來設定 commitlint，沒有本質上的差異

```
pnpm install commitlint @commitlint/cli @commitlint/config-conventional --save-dev -w
```

#### 把 commitlint 集成到husky 中

commit-msg 檔案中可以配置在 git commit 時對 commit 註解的校驗指令可手動創建檔案再輸入檔案內容，但是建議使用命令創建，命令如下:

- npx 是一個 Node.js 工具，用於執行本地或遠端的 npm 包。
- --no-install 表示不自動安裝缺失的依賴，直接使用當前環境中已安裝的版本。
- commitlint 是一個用於檢查 Git 提交訊息格式的工具。
- -e 是 commitlint 的選項，表示從環境變數中讀取提交訊息（通常是 GIT_PARAMS 或 HUSKY_GIT_PARAMS ）。
- $HUSKY_GIT_PARAMS ：
  這是 husky 提供的一個環境變數。
  它指向一個臨時文件，該文件包含了當前提交的提交訊息。
  在 commit-msg Hook 中，這個文件路徑會被傳遞給相關工具（例如 commitlint），以便讀取和檢查提交訊息。
- 棄用原因 ：Husky 7.x 引入了新的配置方式，不再使用 husky add 命令來添加 Git hooks。

```
npx husky add .husky/commit-msg "npx --no-install commitlint -e $HUSKY_GIT_PARAMS"
```
