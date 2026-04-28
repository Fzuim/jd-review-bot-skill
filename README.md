# jd-review-bot

京东批量评价 Claude Code Skill —— 自动给所有待评价商品打五星好评，赚京豆。

## 效果

81 条待评价订单，10 分钟全自动搞定，1000+ 京豆到手。

手动一条条点需要 40 分钟，脚本跑一遍 10 分钟，以后每次只需要一句话。

## 安装

### 1. 安装 Skill

下载 `.skill` 文件后，在 Claude Code 中安装：

```bash
claude skills install jd-review-bot.skill
```

或者直接把本目录放到 Claude Code 的 skills 目录下。

### 2. 安装依赖

Skill 首次运行时会自动安装 `browser-use`，也可以手动安装：

```bash
pip install browser-use
browser-use install
```

### 3. 登录京东

用你的 **真实 Chrome 浏览器** 打开 [jd.com](https://www.jd.com) 并登录账号。

Skill 通过 `--browser real` 复用你的登录 Cookie，不需要输入密码。

## 使用

在 Claude Code 对话中直接说：

> "帮我评价京东的待评价商品"

或者更具体：

> "我京东有一堆待评价订单，全部五星好评"

Skill 会自动：
1. 打开京东评价列表页
2. 收集所有待评价订单
3. 逐个填写好评文字 + 五星评分 + 提交
4. 处理多 SKU 订单和服务评价
5. 完成后验证全部清零

## 原理

### 为什么不用 JS 直接赋值？

京东的评价表单校验逻辑只认真实键盘事件。

`textarea.value = "好评"` 虽然能填进去，但字符计数器显示 `0 / 500`，提交会报「请填写完整的评价内容」。

所以必须用 `browser-use type` 模拟键盘逐字输入。

### 为什么先填文字再评分？

京东的星级评分组件点击后会清空 textarea。

先评分再填文字 → 文字被清空 → 提交失败。

先填文字再评分 → 文字已在 DOM 中 → 提交成功。

### 核心流程

```
收集所有 ruleid（5页）
  ↓
对每个订单：
  ① 先填文字（browser-use type 真键盘输入）
  ② 再评分（.commstar hover + click star5）
  ③ 服务印象（点击正面标签）
  ④ 同步计数器
  ⑤ 点击发表
  ⑥ 验证跳转
  ↓
全部完成，验证清零
```

## 文件结构

```
jd-review-bot/
├── SKILL.md               # Skill 指令
├── README.md              # 本文件
└── scripts/
    └── jd_review.py       # 批量评价脚本
```

## 注意事项

- **必须用 `--browser real --headed`**，否则无法复用登录状态
- 评价文字长度控制在 15-26 字，京东要求 10 字以上才给京豆
- 脚本加了随机延迟（1.5-3秒），避免触发京东反爬
- 只选正面标签（态度好、速度快等），不选负面标签
- 默认开启匿名评价

## License

MIT
