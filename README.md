本仓库规则只作为视频演示用途，考虑到兼容性，通用性，不能满足所有人的需求，请fork到自己仓库根据自己实际情况进行修改。  
  
**稳定，高性价比的专线机场**  
**[机场官网](https://a.qichiyu.com/dog)**

https://github.com/qichiyuhub/rule

## 参考配置

https://raw.githubusercontent.com/qichiyuhub/rule/refs/heads/main/config/mihomo/config/mihomo.yaml

## 常用资源

- OpenClash 内核（meta）：https://github.com/vernesong/OpenClash/tree/core/master/meta
- Koolson/Qure（图标等资源）：https://github.com/Koolson/Qure
- meta-rules-dat（GeoSite/GeoIP）：https://github.com/MetaCubeX/meta-rules-dat/tree/meta/geo

## 秘钥管理

- 公开配置：`config/me/my-configdns.yaml`，不含任何秘钥，可直接提交。
- 私有覆写模块：`config/me/my-secrets.conf`，含订阅链接、账号密码、DoH token，已被 `.gitignore` 忽略。首次使用时复制 `config/me/my-secrets.conf.example` 并填入真实值。
- 在 OpenClash 运行状态页点 **Overwrite Module**，上传 `my-secrets.conf`，匹配配置选 `all`，打开开关后重启插件。
