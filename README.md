
# 中秋 · 月圆心愿 —— 开发总结

> 项目目录：`D:\keshihua\zhongqiu-gaobai\`
> 线上地址：https://4m24yr14xbxd6.aiforce.cloud/app/app_17e9u6auxmm
> 主文件：`index.html`（单文件自包含，72KB，内联全部 CSS/JS）

---

## 一、项目演进

| 阶段 | 定位 | 关键变化 |
|---|---|---|
| v1 | 中秋表白网站 | 浪漫+科技视觉、秋凉深蓝月夜、Canvas 孔明灯、枫叶光标、玉兔精灵 |
| v2 | 公共中秋文化站 | 面向所有人：中秋科普（起源/传说/月相/习俗/诗词）、放孔明灯收集心愿、实时同步计数、心愿墙可单开查看 |
| v3 | 加回复（评论） | 心愿卡片可回复他人，类评论区，云端实时同步 |

## 二、技术架构

- **前端**：原生 HTML/CSS/JS 单文件；字体走 miaoda.feishu.cn 自托管镜像；两张 AI 生成背景图放 `assets/`（hero-bg.jpg、river-bg.jpg）
- **后端**：Supabase（PostgREST REST API，anon key 内置前端）
  - 表 `wishes`：放灯文案（id/text/created_at）
  - 表 `replies`：回复（id/wish_id FK→wishes/text/created_at，RLS：匿名可读可写）
- **托管**：妙搭（lark-cli apps +deploy，app_id `app_17e9u6auxmm`），目录整体打包，无需自备服务器/域名
- **降级策略**：云端不可用时自动回退 localStorage 本地存储，不影响基本放灯流程

## 三、核心功能清单

1. **视觉氛围**：深蓝靛秋夜 + 满月桂花 + 秋凉感；hero/放灯区各一张生成背景
2. **互动细节**：枫叶形鼠标光标（内联 SVG）、可拖动玉兔精灵（点击有反应）
3. **中秋科普**：起源时间轴（先秦→唐→宋→明清）、嫦娥传说、实时月相、六宫格习俗、诗词
4. **放孔明灯**：输入文案（≤24 字）→ Canvas 灯升起动画 → 云端实时计数（"全网已放飞 N 盏"）
5. **心愿墙**：弹窗展示最近 200 条，卡片式布局，可单开查看
6. **回复功能**：每张心愿卡片可回复（≤60 字），按时间序展示，超 5 条折叠
7. **回复区折叠**：点"收起"整条回复区（含文字+输入框）收起，按钮显示"回复 · N"
8. **输入状态保护**：轮询无新数据时不重绘 DOM；有变化时保留展开状态、输入内容并恢复焦点

## 四、踩坑与修复记录（重点）

| # | 问题 | 根因 | 修复 |
|---|---|---|---|
| 1 | LeanCloud 无法注册 | 平台政策变化 | 改用 Supabase（官网国内可达、免实名） |
| 2 | 连接不上 Supabase | Project URL 被误粘成 dashboard 页面网址，且 `?head=true` 不是合法 PostgREST 参数 | 加 URL 归一化函数；计数改 `Prefer: count=exact` |
| 3 | 回复发送后墙上不显示 | PostgREST 嵌套排序语法 `replies(...,order=created_at.asc)` 解析报 PGRST100 400 | 去掉嵌套排序，前端按时间升序 |
| 4 | 回复表刚建好时全链路报错 | PostgREST schema cache 未刷新 | 前端自动降级 + 2 分钟后自动重试嵌入查询 |
| 5 | 点"收起"回复文字不消失 | CSS `display:flex` 覆盖了 `hidden` 属性的 UA 默认 `display:none` | 加 `.replies[hidden]{display:none!important}` |
| 6 | 点"回复"后不操作自动关闭 | 定时轮询重绘整墙，把手动展开状态重置 | 重绘前保存每张卡片展开状态，重绘后恢复 |
| 7 | 轮询后输入中断、焦点丢失 | 重绘重建 input 元素，IME 组合态被打断 | 轮询做签名比对，无数据变化不重绘；必须重绘时恢复输入值并重新聚焦 |



