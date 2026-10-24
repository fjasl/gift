# 依赖与兼容清单

> 一句话：**这个页面没有依赖。**
>
> 下面这份清单不是"我用了什么库"，而是"我把所有可能失效的东西都记录在案，以及它们失效时页面会变成什么样"。

---

## 一、外部依赖

| 类别 | 是否使用 | 说明 |
|---|---|---|
| CDN（jsDelivr / unpkg 等） | **无** | 十年后这些域名可能已经关了 |
| 外部字体（Google Fonts 等） | **无** | 用系统字体栈，`Songti SC` 缺失时自动降级到 Georgia / serif |
| 图标库 / CSS 框架 | **无** | 所有图形都是 CSS 渐变 + 内联 SVG |
| npm 包 | **无** | 页面本身不需要安装任何东西 |
| 构建工具（Webpack/Vite） | **无** | 源文件即产物，改完直接上传 |
| 后端 / 数据库 | **无** | 纯静态 |
| 第三方统计 / 埋点 | **无** | 不会向任何服务器发请求 |

验证方法：打开页面，在开发者工具 Network 面板里看——**除了 `index.html` 本身，应该一条请求都没有**。如果是内联了素材的 `dist/index.html`，连 `assets/` 都不该出现。

---

## 二、运行时能力清单

| API | 用途 | 缺失时的行为 | 降级质量 |
|---|---|---|---|
| `document.*` | DOM 操作 | 进入纯文本降级层 | 必要条件 |
| `setTimeout` | 打字机节奏 | 一次性显示全文 | 无感 |
| `classList` | 状态切换 | 装饰丢失，内容可读 | 可接受 |
| `localStorage` | 未实际使用 | 已用 `try/catch` 包裹 | 无影响 |
| Canvas 2D | 背景尘埃 | 不绘制，剩纯色背景 | 无感 |
| `requestAnimationFrame` | 尘埃动画 | 动画不启动 | 无感 |
| `IntersectionObserver` | 滚动渐显 / 触发打字 | 立即全部显示 | 无感 |
| `matchMedia` | 读"减弱动态效果"偏好 | 视作不需要减弱 | 无感 |
| `HTMLAudioElement` | 背景音乐 | 静音，阅读不受影响 | 无感 |
| `backdrop-filter` | 版本详情卡的毛玻璃 | 变成不透明深色卡片 | 可接受 |
| `clamp()` / `env()` | 字号自适应 / 刘海屏安全区 | 退化为固定字号、多留白 | 可接受 |

**明确不用的东西**（用了会在 `file://` 下坏掉）：

- `fetch` / `XMLHttpRequest` —— 读本地文件会被 CORS 拦截
- ES Module（`import` / `type="module"`）—— `file://` 下同样被拦截
- Service Worker —— 不需要，且要求 HTTPS
- `localStorage` 之外的一切持久化 API

---

## 三、最低可运行环境

| 平台 | 最低版本 | 依据 |
|---|---|---|
| Chrome / Edge | 57 | CSS 自定义属性 + `IntersectionObserver` |
| Firefox | 52 | 同上 |
| Safari（macOS） | 10.1 | 同上 |
| iOS Safari | 11 | `constant()/env()` 安全区 |
| Android Chrome | 60 | 同上 |
| 微信内置浏览器 | 7.x | 内核即 Chrome 57+ |

比这些更旧的浏览器不会白屏——会落入**纯文本降级层**，显示"请用现代浏览器打开"的提示，并告诉你正文在文件里怎么找。

---

## 四、多端适配清单

| 设备 / 环境 | 已处理的问题 |
|---|---|
| **iOS Safari** | 地址栏收放导致的高度跳动（用 `position:fixed` 而非 `100vh`）；刘海与底部横条（`env(safe-area-inset-*)`）；音量属性只读（淡入失效但不报错）；自动播放禁令（由"拆封"点击解锁） |
| **Android Chrome** | 下拉刷新误触（`overscroll-behavior-y:contain`）；双击缩放延迟（`touch-action:manipulation`）；系统字体放大（单列布局，只会变长不会挤坏） |
| **微信内置浏览器** | `backdrop-filter` 加 `-webkit-` 前缀；音频被静音时静默失败；`position:fixed` 抖动（层高控制得比较浅） |
| **WebView（APK）** | `file://` 下的 CORS 限制（零请求设计）；`localStorage` 可能被禁（全部 `try/catch`）；硬件加速合成层数量 |
| **桌面浏览器** | 内容最大宽度 600px 居中；1000px 以上加两侧金色细分隔线；悬停时封面光点增亮；支持 Enter / 空格拆封 |
| **横屏手机** | 封面内容可能超出视口 → 允许纵向滚动，圆环和标题缩小 |
| **小屏（320–360px）** | 内边距收窄、印章缩小、字号下限锁定 |
| **平板** | 走桌面分支，正文不会被拉成一条长线 |
| **打印** | 强制白底黑字，隐藏封面和动画层，版本号转灰 |
| **无障碍** | `prefers-reduced-motion` 关闭全部动画；正文保持可选中复制；标题层级语义化 |

---

## 五、素材内联策略

### 已经内联的

**纸张颗粒**：一段 `feTurbulence` 噪声，写成 SVG data URI 直接嵌在 CSS 里，约 **400 字节**，无双引号转义问题，不产生任何网络请求。

### 为什么 SVG 用 URL 编码而不是 base64

同样的内容，两种写法体积差 **≈33%**：

```
base64:      data:image/svg+xml;base64,PHN2ZyB4bWxucz0i...
URL 编码:    data:image/svg+xml,%3Csvg xmlns='http...%3E
```

base64 对 SVG 是纯浪费——SVG 本身是文本，而 base64 是给**二进制**准备的。所以：

- **SVG / 图标** → `data:image/svg+xml,` + URL 编码
- **JPG / PNG / WebP / MP3** → `data:image/jpeg;base64,`，别无选择

### 你的照片和音乐：用脚本内联

```bash
node tools/inline-assets.mjs
```

它会：

1. 扫描 `index.html` 里所有 `'assets/xxx'` 形式的字符串
2. 读取对应文件，转成 base64 data URI
3. 替换回源码
4. 把 `BUILD` 盖上当天日期
5. 输出到 `dist/index.html`

产物是一个**连图片带音乐全在里面的单个 HTML 文件**，双击就能看，拷到 U 盘、发微信、丢进 APK 的 `assets/` 都能直接用。

### 体积预算

| 内容 | 原始 | base64 后 |
|---|---|---|
| 音乐 `duvet.mp3`（3:24，80kbps） | 1.94 MB | 2.59 MB |
| 照片 6 张 × 150KB（待加） | 900 KB | 1.2 MB |
| 页面本身 | 35 KB | 35 KB |
| **合计** | **≈2.9 MB** | **≈3.8 MB** |

当前的 `dist/index.html` 是 2.62 MB（只有音乐，还没有照片），4G 下约 2 秒。

**音乐的压缩参数**（`ffmpeg`，用 `-map_metadata -1` 顺手去掉了内嵌封面图，否则光封面就几十 KB）：

```bash
ffmpeg -i duvet.mp3 -map_metadata -1 -b:a 80k -ac 2 -ar 44100 assets/duvet.mp3
```

80kbps 对手机外放、低音量循环播放足够了；如果要塞更多照片，可以把音乐降到 64kbps 单声道，还能再省 30%。

2.5MB 在 4G 下约 2 秒，可以接受。**超过 4MB 就该压图了**，脚本会主动警告。

---

## 六、十年后怎么打开

### 最短路径

双击 `index.html`。不需要服务器，不需要联网，不需要装任何东西。

### 如果只看到一段提示文字

说明 JS 被拦或浏览器太旧。**内容本身没坏**——正文就在文件里，用记事本打开搜索 `CONFIG`，那几段中文就是信的全文。

### 备份策略

| 去哪 | 为什么 |
|---|---|
| Git 仓库 | 有历史，改坏了能回退 |
| U 盘 / 移动硬盘 | 不依赖任何在线服务 |
| 云盘 | 防本地硬盘损坏 |
| 打印一份 | 纸是最抗时间的技术，页面已适配打印样式 |

`.html` 是纯文本格式，和 .txt 一样不会腐坏；真正会消失的是域名、CDN、云服务和你的账号。所以**把文件本身当成礼物，而不是把链接当成礼物**——链接是那个 APK 版本存在的理由。

### 自己验证"还能不能跑"

1. 断网，双击打开 → 能正常拆封、读完
2. 看 Network 面板 → 应该只有一条 `index.html` 请求
3. 长按底部版本号 → 弹出构建详情，确认版本对得上

三项都过，这份东西就再活十年。
