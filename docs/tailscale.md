# Tailscale 支持（本 Fork 新增）

上游 starifly/NekoBoxForAndroid 的 libcore 没有注册 sing-box 的 Tailscale
endpoint，核心里也没有编译。本 Fork 做了三处改动：

- `libcore/box_include.go`：注册 `tailscale.RegisterEndpoint` 和
  `tailscale.RegistryTransport`（MagicDNS 的 tailscale DNS 传输）
- `libcore/build.sh`：gomobile 构建标签加上 `with_tailscale`
  （sing-box 没有这个标签时 Tailscale 是桩实现，运行时会报错）
- `libcore/go.mod` / `go.sum`：补上 tailscale 相关依赖

注意：DERP 服务（自建中继）没有注册，手机做客户端用不到。

## 用法

重新编译安装后，在 App 里新建「自定义配置」，或在设置里的
「全局自定义配置」中加入下面的 endpoint 和路由规则片段。
把去 tailnet 的流量（100.64.0.0/10 和 *.ts.net）交给这个 endpoint，
其余流量仍走你原来的代理节点。

```json
{
  "endpoints": [
    {
      "type": "tailscale",
      "tag": "ts",
      "auth_key": "tskey-auth-REPLACE_ME_LOCALLY",
      "hostname": "nekobox-phone",
      "accept_routes": true
    }
  ],
  "route": {
    "rules": [
      { "ip_cidr": ["100.64.0.0/10"], "outbound": "ts" },
      { "domain_suffix": ["ts.net"], "outbound": "ts" }
    ]
  }
}
```

## 安全警告

- `auth_key` 是加入 tailnet 的凭证，**只能在本机填写，永远不要提交进
  本仓库**（本 Fork 是公开的，提交等于公开泄露，任何人都能用它加入
  你的 tailnet）。
- auth key 在 Tailscale 控制台 Settings → Keys 生成，建议关掉
  Ephemeral、开 Reusable，并设置合理过期时间；泄露后立即在控制台
  Revoke 并重新生成。
- 上面的模板不含任何真实密钥。
