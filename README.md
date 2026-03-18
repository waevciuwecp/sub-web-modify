# sub-web-modify
[本项目](https://suburl.v1.mk)重制[原项目](https://github.com/CareyWang/sub-web)CSS样式，兼容nodejs最新版本（可直接一键部署至Vercel），解决大部分布局细节问题，增加“暗黑模式”，默认自动切换亮/暗模式（点击“太阳/月亮”图标可手动切换），增加“高级功能”点击显示/隐藏，添加短链接选择/自定义功能，增加近百条远程配置，新增[sub-web聚合API](https://github.com/youshandefeiyang/sub-web-api)，增加从短链接中获取订阅信息并返回至前端界面，增加上传自定义远程配置/JS进阶排序节点/JS进阶筛选节点等功能，感兴趣的朋友可以自建API服务，增加URL传参设置自定义后端<br/>
## 效果预览：
![avatar](https://raw.githubusercontent.com/youshandefeiyang/webcdn/main/sub-web-modify.GIF)
### 使用方法：
建议使用Docker一键部署:
```
docker run -d --restart unless-stopped --privileged=true -p 8090:80 --name sub-web-modify youshandefeiyang/sub-web-modify
```
访问地址举例:
```
http://192.168.10.1:8090/?backend=https://url.v1.mk
```

## Dialer / Proxy Providers（新功能）

### 1) 远程配置新增
- `Dialer`（默认，非负载均衡）  
  `https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer-non_lb.ini`
- `Dialer LoadBalance`（可选）  
  `https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer.ini`
- `Dialer LB Media`（provider 直连媒体）  
  `https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer.direct-media.ini`

页面中“使用 Dialer 远程配置（默认）”按钮会直接切换到默认 `Dialer` 配置。

### 2) 后端地址新增
在“后端地址”中新增以下预设：
- `https://sub-dialer.yaoyy.moe`
- `http://100.91.70.43:25501`（tailscale-dialer）
- `http://192.168.194.245:25501`（zerotier-dialer）

### 3) 主表单默认值（UI 重构后）
- 应用标题更新为 `Awesome idea front`
- “订阅命名”默认值：`default.yaml`
- “更新间隔”默认值：`2`（天）

### 4) Providers 输入与单框预览（引导 + Native JSON）
“Dialer Proxy Providers”支持双输入模式：

1. 左侧引导输入（逐条添加）
2. 右侧 Native JSON 单框编辑（可直接粘贴/修改 JSON）

功能按钮：
- “验证并回填”：校验右侧 JSON 后写回左侧引导表单
- “刷新预览”：将左侧当前值格式化回右侧 JSON
- 支持“填入示例”和“清空 Providers”

同步行为：
- 左侧引导输入变化会实时同步到右侧 JSON
- 右侧 JSON 输入变化会实时更新匹配统计

### 5) 远程 ini 预取与表达式匹配高亮
- 当前 `远程配置` 变更后，前端会自动预取对应 ini
- 自动识别 `custom_proxy_group=...` 中与 dialer 相关且 `select-use` 的表达式
- 在预览区顶部展示“允许表达式”
- 在右侧 JSON 单框内对 `name` 字段匹配片段进行高亮
- 匹配统计以 `matched / total` 显示
- 若远程 ini 受 CORS/网络限制导致失败，会显示错误提示并跳过高亮匹配

### 6) 从URL解析支持 Dialer / Sing-box 参数
“从URL解析”已支持以下参数回填到表单：
- `use_dialer` / `useDialer`
- `dialer_group_name` / `dialerGroupName`
- `apply_dialer_to` / `applyDialerTo`
- `proxy_providers` / `proxy-providers` / `proxyProviders`
- `singbox_ver` / `singbox.ver` / `ver`（当 `target=singbox`）

其中 `use_dialer` 支持 `true/1/yes`。
`singbox_ver` 在前端按后端最新约束校验为 `1.12.x - 1.14.x`。

### 7) proxy_providers 编码说明
- 前端按标准 `encodeURIComponent` 发送 `proxy_providers`
- 若 provider 的 URL 内本身含 `%3A/%2F/...` 等编码，当前实现可保持其编码语义，不会在输出 YAML 中被错误展开

### 8) provider JSON 示例（后端参数格式）
当你需要直接拼 URL 参数时，`proxy_providers` 对应的是 JSON 数组，例如：
```json
[
  {"name":"provider-a","type":"http","url":"https://example.com/a.yaml"},
  {"name":"provider-b","type":"http","url":"https://example.com/b.yaml","interval":3600}
]
```

### 9) Digest 短链接 / 二维码优化
- Digest 模式会优先使用紧凑模式 `m=1`（短键 + 布尔位图 `bt/bf`）组织 `q`，并在以下候选中自动选择最短结果：
  - `deflateRaw + base64url(紧凑 q)`
  - `base64url(紧凑 q)`
  - `deflateRaw + base64url(完整 q)`
  - `base64url(完整 q)`
- 常用短键映射：`t->target`、`u->url`、`c->config`、`i->include`、`e->exclude`、`r->rename`、`d->dev_id`、`iv->interval`、`p->proxy_providers`、`v->ver`、`sv->singbox_ver`、`dg->dialer_group_name`、`da->apply_dialer_to`。
- Digest 模式下，`filename` 不再写入 `q`，只通过 `a` 传递，避免重复参数导致链接变长；全量参数名仍支持手工编辑。
- 从 URL 解析时，若路径是 `/digest` 且存在 `a`，前端会优先使用 `a` 回填“订阅命名”。
- 二维码生成使用更低纠错等级（`L`）与更大尺寸（按 URL 长度自适应），提升长链接的扫码成功率。

### 10) GitHub Actions 运行时
- Docker 构建工作流已升级到 Node 24 兼容版本：
  - `actions/checkout@v5`
  - `docker/setup-buildx-action@v4`
  - `docker/login-action@v4`
  - `docker/build-push-action@v7`
