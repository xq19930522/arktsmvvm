# ArkTS MVVM Boilerplate 使用文档

一款用于 HarmonyOS / DevEco Studio 的 IDEA 插件：在任意目录右键，一键生成符合官方架构规范的 ArkTS MVVM 三层骨架代码（View / ViewModel / Model）。

- 插件版本：1.1.0
- 兼容平台：IntelliJ Platform 223 (2022.3) ~ 252.*，覆盖 DevEco Studio 3.1+ / 4.x / 5.x

---

## 一、安装插件

### 方式 1：从本地 zip / jar 安装（开发版）

1. 打开 DevEco Studio，菜单栏选择 **File → Settings**（macOS 为 **DevEco Studio → Preferences**）
2. 左侧选择 **Plugins**，点击右上角 ⚙ 齿轮图标，选择 **Install Plugin from Disk...**
3. 选择 `arktsmvvm-1.1.0.zip`（或 `arktsmvvm-1.1.0.jar`），点击 **OK**
4. 点击 **Restart IDE** 重启使插件生效

### 方式 2：从 JetBrains Marketplace 安装（发布后）

1. **File → Settings → Plugins → Marketplace**
2. 搜索 `ArkTS MVVM Boilerplate`，点击 **Install**
3. 重启 IDE

---

## 二、使用步骤

### 1. 选择目标目录

在 Project 视图中，**右键点击任意目录**（例如 `ets/pages/` 下新建的功能文件夹）：

- 如果右键的是文件，插件会自动使用它所在的目录
- 菜单选择 **New → ArkTS MVVM Boilerplate**

### 2. 填写类名

在弹出的对话框中输入**类名**（合法的 ArkTS 标识符：字母 / 数字 / 下划线，不能以数字开头），非法输入会实时红字提示。

对话框选项：

| 选项 | 说明 |
|---|---|
| **Generate as Page**（默认勾选） | 生成的 View 带 `@Entry`，是一个独立页面，**需要注册到 `main_pages.json`** |
| 取消勾选 | 生成的 View 是纯 `@ComponentV2` 组件，可在任意页面中 `import` 使用 |

### 3. 确认生成

点击 **OK**，插件会在目标目录下生成三个文件：

```
你右键的目录/
├── view/
│   └── XxxView.ets        # View 层：纯 UI
├── viewmodel/
│   └── XxxViewModel.ets   # ViewModel 层：状态与业务
└── model/
    └── XxxModel.ets       # Model 层：数据模型与数据源
```

> 如果同名文件已存在，插件会先弹出确认框，选择 **Yes** 才会覆盖。

生成完成后会弹出成功提示。整个生成操作**支持 Ctrl+Z 撤销**。

### 4. 页面需要注册路由（仅 Generate as Page 时）

如果勾选了 "Generate as Page"，需要在工程中找到路由配置文件并添加页面路径：

```json
// src/main/resources/base/profile/main_pages.json
{
  "src": [
    "pages/Index",
    "pages/你的目录/XxxView"   // ← 添加这一行（不带 .ets 后缀）
  ]
}
```

然后通过 `this.pageInfos.pushPath({ name: '你的目录/XxxView' })` 等方式跳转。

---

## 三、生成代码说明

### Model 层（`XxxModel.ets`）

- 定义 `XxxData` 数据接口（默认含 `id / title / description` 三个字段，自行修改）
- 单例模式的 `XxxModel` 类，内置模拟异步数据源的示例方法：
  - `fetchXxxList()`：拉取列表
  - `fetchXxxDetail(id)`：按 id 查详情
- 实际项目中把模拟实现替换为 Http / RCP / Preferences / RelationalStore 即可

### ViewModel 层（`XxxViewModel.ets`）

采用**状态管理 V2**：

```typescript
@ObservedV2
export class XxxViewModel {
  @Trace public xxxList: XxxData[] = [];   // 列表数据
  @Trace public isLoading: boolean = false; // 加载状态
  @Trace public errorMessage: string = '';  // 错误信息
}
```

- 被 `@Trace` 标注的属性变化会被 View 自动感知
- `loadXxxList()`：完整的 加载中 → 成功 / 失败 → 结束 状态流转示例

### View 层（`XxxView.ets`）

```typescript
@Entry              // 仅 Generate as Page 时有此行
@ComponentV2
export struct XxxView {
  @Local private viewModel: XxxViewModel = new XxxViewModel();
```

- 通过 `@Local` 订阅 ViewModel，`@Trace` 属性变化自动刷新 UI
- 自带完整 UI 流程：加载中（`LoadingProgress`）→ 错误提示 → 列表展示（`List + ForEach`）
- View 只与 ViewModel 交互，**不直接访问 Model**，符合官方 MVVM 规范

---

## 四、完整示例

输入类名 `Book`，勾选 "Generate as Page"，生成：

```
view/BookView.ets
viewmodel/BookViewModel.ets
model/BookModel.ets
```

注册路由后运行，即可看到一个自动加载模拟数据的列表页面，可直接在此基础上改字段、接真实接口。

---

## 五、常见问题

**Q1：生成的页面运行后白屏 / 跳转报错？**
勾选 "Generate as Page" 时生成的 View 带 `@Entry`，必须注册到 `main_pages.json`，详见第二章第 4 步。

**Q2：想生成子组件而不是页面？**
取消勾选 "Generate as Page"，然后在页面中：

```typescript
import { BookView } from './你的目录/view/BookView';

build() {
  Column() {
    BookView()
  }
}
```

**Q3：覆盖了已有文件能恢复吗？**
覆盖前插件会弹窗确认。如果已经覆盖，立即按 **Ctrl+Z** 可以撤销生成操作。

**Q4：右键菜单里找不到 "ArkTS MVVM Boilerplate"？**
确认插件已安装并重启 IDE；该菜单项位于 **New** 子菜单中（与 New File / New Directory 同级）。

---

## 六、版本历史

### 1.1.0（当前版本）

- 模板升级至状态管理 V2：`@ObservedV2` / `@Trace` / `@ComponentV2` / `@Local`
- 新增 "Generate as Page" 选项：页面（`@Entry`）/ 纯组件二选一
- 生成操作支持撤销（Ctrl+Z）
- 覆盖已有文件前增加确认提示
- 对话框内实时类名校验

### 1.0.0

- 首个版本：生成 ArkTS MVVM（View / ViewModel / Model）三层骨架代码
