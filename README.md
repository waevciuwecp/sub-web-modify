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
  `https://raw.githubusercontent.com/waevciuwecp/subconverter_asdlokj1qpi233/refs/heads/master/base/config/nodnsleak.dialer-non_lb.ini`
- `Dialer LoadBalance`（可选）  
  `https://raw.githubusercontent.com/waevciuwecp/subconverter_asdlokj1qpi233/refs/heads/master/base/config/nodnsleak.dialer.ini`

页面中“使用 Dialer 远程配置（默认）”按钮会直接切换到默认 `Dialer` 配置。

### 2) 后端地址新增
在“后端地址”中新增以下预设：
- `https://sub-dialer.yaoyy.moe`
- `http://100.91.70.43:25501`（tailscale-dialer）
- `http://192.168.194.245:25501`（zerotier-dialer）

### 3) Providers 输入体验优化
“Dialer Proxy Providers”不再要求手写整段 JSON，改为逐条添加：
1. 点击“新增一条 Provider”
2. 填写 `name`、`url`（必填）
3. 可选填写 `type`、`path`、`interval`
4. 继续新增下一条

支持“填入示例”和“清空 Providers”。

### 4) 从URL解析支持 Dialer 参数
“从URL解析”已支持以下参数回填到表单：
- `use_dialer` / `useDialer`
- `dialer_group_name` / `dialerGroupName`
- `apply_dialer_to` / `applyDialerTo`
- `proxy_providers` / `proxy-providers` / `proxyProviders`

其中 `use_dialer` 支持 `true/1/yes`。

### 5) proxy_providers 编码说明
- 前端按标准 `encodeURIComponent` 发送 `proxy_providers`
- 若 provider 的 URL 内本身含 `%3A/%2F/...` 等编码，当前实现可保持其编码语义，不会在输出 YAML 中被错误展开

### 6) provider JSON 示例（后端参数格式）
当你需要直接拼 URL 参数时，`proxy_providers` 对应的是 JSON 数组，例如：
```json
[
  {"name":"provider-a","type":"http","url":"https://example.com/a.yaml"},
  {"name":"provider-b","type":"http","url":"https://example.com/b.yaml","interval":3600}
]
```
