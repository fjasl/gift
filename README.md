# 给你的信 · 使用说明

一个零依赖的单文件页面，同时可作**网页托管**和**APK 内置页面**两种形态发布。

```
gift/                  ← 源文件，日常改这个
├── index.html
├── assets/
│   └── duvet.mp3
├── DEPENDENCIES.md
└── CHANGELOG.md

dist/                  ← 构建产物，托管和打包 APK 用这个
└── index.html         ← 图片和音乐全部内联，约 2.6 MB

tools/
├── inline-assets.mjs  ← 把照片/音乐转 base64 内联，产出 dist/index.html
└── shoot.mjs          ← 用 CDP 拍真实手机尺寸的截图（含点击后动画）
```

> **一句话记住**：`gift/` 是草稿，`dist/` 是成品。每次改完 `gift/index.html` 都要重新跑一次构建，否则线上还是旧版。

---

## 一、改内容

所有可改的东西都集中在 `index.html` 里 `<script>` 顶部的 `CONFIG` 对象，**不要动下面的渲染逻辑**。

| 字段 | 作用 | 示例 |
|---|---|---|
| `to` | 收件人，写在信封正面，也是页面标题 | `'给林小满'` |
| `from` | 落款 | `'阿舟'` |
| `dateStart` | 在一起的日子。填了才会显示"在一起的第 N 天"，留空则隐藏 | `'2023-05-20'` |
| `lead` | 开场白，`\n` 会换行 | 见文件内默认值 |
| `letter` | 信的正文，**数组，每个元素是一段** | 见文件内默认值 |
| `photos` | 照片列表 | `[{src:'assets/01.jpg', cap:'那天下了雨'}]` |
| `music` | 音乐文件路径，留空则不显示音乐按钮 | `'assets/duvet.mp3'` |
| `musicVolume` | 音量，0~1 | `0.28` |
| `sealText` | 结尾印章里的字 | `'印'` |

> 音乐按钮从**首屏（信封那一页）**就在右下角，不用等拆封。每个版本首次打开会压一层灰色蒙版、
> 用折线箭头引导一次；轻触任意处关掉，之后按钮回到很淡的待机状态。

正文里可以用三个占位符：`{days}`（第几天）、`{to}`、`{from}`，会自动替换。

> 默认文案是按"贴在贴纸上的信"这个设定写的，如果你最终只发网页链接（对方没碰贴纸），把 `letter` 第一段改掉即可。

---

## 二、版本声明

这是你特别要求的部分，页面里有**四层**版本可查：

| 位置 | 形式 | 用途 |
|---|---|---|
| `<head>` 里 | `<meta name="gift-version" content="1.0.0">` | 爬虫/工具读取 |
| 页面底部 | `v1.0.0 · 2026-02-14` | 肉眼可见 |
| 控制台 | 彩色横幅打印版本号 | 调试时一眼看到 |
| `window.GIFT` | `{version, build, config}` | 脚本化检查线上版本 |

**发布新版本时改这两个常量**（在 `<script>` 最上面）：

```js
var VERSION = '1.0.0';      // 语义化版本
var BUILD   = '';           // 填 'YYYY-MM-DD'，留空则只显示版本号
```

### 两个实际用途

**① 确认线上跑的是不是最新版**
页面底部文字**长按 0.8 秒**，会弹出构建详情：版本号、构建日期、域名、协议、加载耗时、打开时间。部署完对一下就知道有没有传错。

**② 破缓存**
手机浏览器和 WebView 都会激进缓存 HTML。改完内容上传后，如果还看到旧版：

- 在链接后加参数：`https://xxx.com/?v=1.0.1`（NFC 贴纸里可以直接写带参数的地址）
- 或者直接把 `VERSION` 加在文件名上：`index.html` → `index-v101.html`

**每次改了页面内容，就把 `VERSION` 递增一位，并在 `CHANGELOG.md` 里记一行。** 半年后再改，你会谢自己。

---

## 三、形态一：网页托管

### 腾讯云 EdgeOne Pages

1. 控制台搜索 "Pages" 或 "静态网站托管"
2. 新建项目 → 选**直接上传文件**（不用连 Git）
3. 把 `gift/` 整个文件夹拖进去，保证 `index.html` 在**根目录**
4. 拿到平台给的默认域名（HTTPS 自动配好）
5. 手机浏览器直接打开验证

注意：

- 需要**实名认证**
- 测试阶段**不要绑自定义域名**，那要 ICP 备案，会卡几天
- `assets/` 目录要一起上传，否则照片 404（页面会显示"图片未找到"占位，不会白屏——这是刻意设计的）

### 本地预览

在 `gift/` 目录里执行（任选其一）：

```bash
python -m http.server 8000
npx serve .
```

手机连同一个 Wi-Fi，访问 `http://电脑IP:8000`。

---

## 四、形态二：APK

这个页面**从第一行就是按"能塞进 APK"写的**，所以迁移成本几乎为零：

| 设计约束 | 为什么 |
|---|---|
| 单文件、零外部请求 | `file://` 下 `fetch`/XHR 会被 CORS 拦，ES module 也加载不了 |
| 普通 `<script>`，不是 `type="module"` | 同上 |
| 照片用相对路径 `assets/01.jpg` | 放进 `assets/` 目录后路径不变 |
| `localStorage` 全部 try/catch | WebView 里可能被禁用，不能让页面崩 |
| 无 Service Worker | 不需要，也不需要 HTTPS 前提 |

接入只需三步：

1. Android 工程里把 `gift/` 下的 `index.html` 和 `assets/` 整个放进 `app/src/main/assets/`
2. `WebView.loadUrl("file:///android_asset/index.html")`
3. WebSettings 打开 `javascriptEnabled`、`domStorageEnabled`、`mediaPlaybackRequiresUserGesture` 保持默认（我们的音频由点击触发，天然合规）

```kotlin
webView.settings.apply {
    javaScriptEnabled = true
    domStorageEnabled = true
    allowFileAccess = true
    // 不需要 allowFileAccessFromFileURLs —— 页面没有任何 file:// 的 XHR
}
webView.loadUrl("file:///android_asset/index.html")
```

> APK 形态的好处是不依赖网络、不会因为域名过期而失效；代价是安装时要绕过"未知来源"提示。**建议两种都给**：网页链接写进 NFC 贴纸（方便、可更新），APK 作为备份（长期可靠）。

---

## 五、照片与音乐规范

**照片**

- 长边压到 **1080px** 以内，转 WebP 或质量 80 的 JPEG
- 单张控制在 **150KB** 以内，总大小 **2MB 以内**
- 命名 `01.jpg`、`02.jpg`…，在 `CONFIG.photos` 里按顺序登记
- 竖图横图都能放，容器是自适应高度的

**音乐**

- `mp3` 或 `m4a`，**60–90 秒**足够（循环播放，太长会烦）
- 128kbps 即可，体积约 1MB
- 用淡入（页面已内置，2 秒渐到 50% 音量）
- 注意：iOS 上 `volume` 属性是只读的，淡入会失效但不会报错，属于可接受的降级

---

## 六、把素材内联进来（单文件交付）

照片和音乐放在 `assets/` 里是"多文件"，托管没问题，但不利于长期保存——十年后 `assets/` 目录可能在某次整理中被删掉。

想彻底变成一个文件，在项目根目录执行：

```bash
node tools/inline-assets.mjs
```

脚本会读取 `index.html` 里所有 `'assets/xxx'` 形式的引用，把文件转成 base64 内联进去，并盖上当日 `BUILD` 日期，输出 `dist/index.html`。

产物**双击就能看，断网也能看，拷进 U 盘、发微信、丢进 APK 的 assets 都能直接用**。

脚本还会顺手把 `BUILD` 改成当天日期，所以信纸底部那行版本号会自动变成 `v2.1.0 · 2026-10-24` 这种形式，一眼能看出是哪天构建的。

代价是体积：base64 会让二进制膨胀约 33%。所以照片务必先压到长边 1080px、单张 150KB 以内。

> 页面里已经内联了一段 SVG 噪声做纸张颗粒（约 400 字节）。SVG 用 URL 编码比 base64 小 33%，这是刻意的选择，原因写在 [DEPENDENCIES.md](DEPENDENCIES.md) 第五节。

---

## 七、多端适配

完整的适配清单见 [DEPENDENCIES.md](DEPENDENCIES.md) 第四节，这里只说要点：

| 环境 | 关键处理 |
|---|---|
| iOS Safari | 用 `position:fixed` 而非 `100vh`，避开地址栏收放跳动；`env(safe-area-inset-*)` 处理刘海和底部横条 |
| Android Chrome | `overscroll-behavior-y:contain` 禁掉下拉刷新，别让一次下拉把信刷没了 |
| 微信内置浏览器 | 补 `-webkit-backdrop-filter`；音频被静音时静默失败而不是报错 |
| WebView / APK | `file://` 下零请求设计；`localStorage` 全部 `try/catch` |
| 桌面 | 正文 600px 居中，1000px 以上两侧加金色细分隔线，支持 Enter 拆封 |
| 横屏 / 矮屏 | 封面允许纵向滚动，圆环和标题缩小，不会卡住看不到"拆封" |
| 小屏 320px | 内边距收窄、印章缩小、字号锁下限 |
| 打印 | 白底黑字，隐藏封面和动画层 |

另外还有一层兜底：**如果 JS 被拦截或执行出错，页面会切换到纯文本降级层**，显示"请用现代浏览器打开"，而不是一片白屏。

---

## 八、上线前检查清单

- [ ] 改了 `to` / `from` / `letter`，没有残留示例文字
- [ ] `dateStart` 要么填对，要么留空（填错会显示错误的天数）
- [ ] 照片真实存在，路径大小写一致
- [ ] **iOS 和 Android 各试一次**（不是同一套渲染）
- [ ] 微信里打开一次（NFC 不经过微信，但链接可能被转发）
- [ ] 手机横过来一次，确认封面还能看到"轻触，拆封"
- [ ] 4G 网络下打开，计时 < 3 秒
- [ ] 关掉网络再打开一次，确认页面不依赖任何外部请求
- [ ] 音量键调到中间，确认音乐不是突然炸出来
- [ ] 从 NFC 贴纸完整走一遍：碰 → 打开 → 拆封 → 读完
- [ ] `VERSION` 已递增，`CHANGELOG.md` 已记录
- [ ] 确认线上版本号与本地一致（长按底部版本号）
