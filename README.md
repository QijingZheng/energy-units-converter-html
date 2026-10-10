# 能量单位换算器（网页版）

`index.html` —— 单文件静态页面，由 [`energy_unit_conv.py`](https://github.com/QijingZheng/pyband/blob/master/energy_unit_conv.py) 改写而来。

界面与文档**中英双语**，默认英文，右上角按钮切换。

**线上地址**：<http://staff.ustc.edu.cn/~zqj/assets/misc/EnergyUnitsConverter/>

## 部署

只需 **`index.html` 一个文件**，无需任何服务器端支持（不需要 PHP/Python/数据库），本地双击也能直接用（`file://` 下功能完全正常）。

当前部署在个人主页的 Jekyll 站点下，源文件放在 **Jekyll 源码根目录**的 `assets/misc/EnergyUnitsConverter/index.html`：

```
jekyll-源码/
├── _config.yml
├── _site/                                    ← 构建产物，不要往这里放
└── assets/misc/EnergyUnitsConverter/
    └── index.html                            ← 放这里
```

更新页面后同步并重新构建：

```bash
cp index.html  <jekyll-源码>/assets/misc/EnergyUnitsConverter/
cd <jekyll-源码> && bundle exec jekyll build
cmp index.html _site/assets/misc/EnergyUnitsConverter/index.html   # 确认是原样拷贝
```

要点：

- 文件**不能**直接放进 `_site/`，`jekyll build` 会先清空该目录。
- `index.html` **不要加 YAML front matter**。Jekyll 只加工带 front matter 的文件，没有 front matter 的会作为静态文件原样拷贝——这正是我们要的，加了反而会被套进主题 layout 导致页面嵌套错乱。
- 页面完全自包含（除 KaTeX 的 CDN 外没有站内相对链接），所以 `_config.yml` 的 `baseurl` 设成什么都不影响，无需改配置。
- 在 Markdown 里引用时用 `relative_url` 过滤器，不要用 `{% link %}`——后者只能指向文件、无法指向目录：

  ```liquid
  [能量单位换算器]({{ '/assets/misc/EnergyUnitsConverter/' | relative_url }}){:target="_blank" rel="noopener"}
  ```

- `README.md`、`energy_unit_conv.py` 都不需要上传。

## 功能

- 12 个单位实时联动：eV、meV、Hartree(a.u.)、Ry、J、kJ/mol、kcal/mol、cm⁻¹、K、THz、nm、fs
- 中英双语切换：**默认显示英文**（方便国外访客），点「中文」切换，选择记入 `localStorage` 下次沿用
- 有效数字 4–12 位可调；自动 / 普通 / 科学记数法
- 每栏可一键复制，或「复制全部」复制成对齐文本表
- 分享链接：输入时地址栏会变成 `#eV=1.5`，把链接发给别人即可复现该输入
- 浅色 / 深色主题（默认跟随系统，可手动切换并记住）
- 手机 / 平板 / 电脑自适应（卡片按宽度自动 1–4 列）
- 页面内文档：换算公式、常数表、常用换算因子速查、约定说明

## 实现要点

- **常数**：代码里只硬编码 SI 精确定义值（h、e、c、k_B、N_A）与 CODATA 2022 的 E_h，其余换算因子全部由它们推导；文档区的常数表、速查表由同一组值动态生成，不会和计算逻辑脱节。
- **换算**：以 eV 为中枢，每个单位一对 `toEv` / `fromEv`，与原 Python 脚本结构一致。
- **双语**：静态 HTML 以英文书写，启动时把英文原文采集进字典，所以英文只写一遍、不会与译文脱节；中文译文集中在 JS 的 `STR.zh` 里。切换语言时会重跑 `fillTables()` 和 `renderMath()`——文档第四节的 `<li>` 内嵌 KaTeX 公式，换 `innerHTML` 会把已渲染的公式抹掉，必须重渲染。
- **公式排版**：优先用 KaTeX（jsDelivr CDN，版本锁定 0.16.9 + SRI 校验）；CDN 不可达、断网或校验失败时自动退回页面里已有的纯 HTML/Unicode 公式，换算功能不受影响。页面「文档」小标题处会显示当前用的是 KaTeX 还是纯文本。

## 自检

地址栏追加 `#selftest` 打开页面（如 `index.html#selftest`），顶部会列出 30 项数值断言的结果（中英文各一份）：
与 CODATA 基准值比对（1 eV = 8065.543937 cm⁻¹ = 11604.51812 K = 96.48533212 kJ/mol = …，
1 Ha = 27.211386245981 eV = 219474.6313632 cm⁻¹ = …）、各单位往返一致性、E ≤ 0 时 nm/fs 无定义。

## 与原 Python 脚本的差异

| | 脚本 | 网页 |
|---|---|---|
| 常数 | 2018 年以前的 CODATA（`h = 4.13566733e-15 eV·s`，`e = 1.60217733e-19 C`） | 2019 年 SI 精确定义值 + CODATA 2022 |
| 单位 | 7 个 | 12 个（增加 meV、Hartree、Ry、J、kcal/mol） |

脚本原始出处：<https://github.com/QijingZheng/pyband/blob/master/energy_unit_conv.py>

两者结果在第 7–8 位有效数字起有差别（例如 1 eV = 96.4854 vs 96.48533 kJ/mol），属常数版本更新所致。
如果希望脚本与网页完全一致，把 `energy_unit_conv.py` 顶部的常数换成 `index.html` 中 `var C = {...}` 的取值即可。

## 修改

所有 CSS 和 JS 都内联在 `index.html` 里：

- 增删单位 → 改 `var UNITS = [...]`（给出 `toEv` / `fromEv` 一对函数、以及 `name:{en,zh}`，页面卡片、公式表和速查表会自动跟着生成；公式表第三列另在 `FORMULA` 里给 TeX 与纯 HTML 两种写法）
- 改文案 → 英文直接改 HTML 里带 `data-i18n` 的那段；中文改 JS 的 `STR.zh`，键名与 `data-i18n` 一致
- 改常数 → 改 `var C = {...}`，派生量 `D` 和文档表格自动更新
- 改配色 → 改 `:root{ --bg: ... }` 及其下的深色覆盖块
