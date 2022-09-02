---
layout: article
title: Vault-Consul-AK\SK动态切换
tags: Project
category: blog
date: 2022-09-02 000000 +0800
mermaid: true
---
## Vault

```bash
wget https://releases.hashicorp.com/vault/1.11.2/vault_1.11.2_linux_amd64.zip
unzip -d /usr/local/bin vault_1.11.2_linux_amd64.zip
```

```bash
vault server -dev -dev-listen-address=0.0.0.0:8200 
```
新开终端
```bash
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="hvs.l6LaNr28TufGp3pyoqOrIG6a"
vault status -address='http://127.0.0.1:8200'
```
```bash
vault secrets enable alicloud
```

- 在阿里云中创建一个自定义策略 ，用于您将提供给 Vault 的访问密钥。请参阅“Example RAM Policy for Vault”。

- 在阿里云中创建一个名为“hashicorp-vault”的用户，并在“用户授权策略”部分直接将新的自定义策略应用到该用户，我直接使用最高权限。

- 在阿里云中为该用户创建访问密钥，这是用户页面上阿里云 UI 中可用的操作。


![在这里插入图片描述](https://img-blog.csdnimg.cn/b37bf7c8d5a94f2a819bde746af1bfbf.png)

```bash
vault write alicloud/config \
    access_key= \
    secret_key=
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/81b5d8ab68ca4a23ac19cfb3e595393c.png)

配置一个角色，描述如何授予凭据。仅使用已在阿里云中创建的策略生成访问令牌。

```bash
vault write alicloud/role/policy-based \
    remote_policies='name:AdministratorAccess,type:System' 
```
仅使用将由 Vault 在阿里云中动态创建的策略生成访问令牌：

```bash
vault write alicloud/role/policy-based \
    inline_policies=-<<EOF
[
    {
        "Statement": [
            {
                "Action": "*",
                "Effect": "Allow",
                "Resource": "*"
            }
        ],
        "Version": "1"
    }
]
EOF
```

```bash
vault write alicloud/role/dev-role   role_arn='acs:ram::1578287549522794:role/dev-role'
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e2cf219cefd8444eabd88d48323daa6e.png)

**生成密钥**

在配置秘密引擎并且用户/机器拥有具有适当权限的 Vault 令牌后，它可以生成凭据。
/creds通过从具有角色名称的端点读取来生成新的访问密钥：

```bash
vault read alicloud/creds/policy-based
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/325a26cc40be458f95d7a3825e62c754.png)

**回收密钥**

```bash
vault lease revoke alicloud/creds/policy-based/KSPlpAF9wwoc88Pg4wvVZyKg
```
可以看到生成了新的用户
![在这里插入图片描述](https://img-blog.csdnimg.cn/b51a622e2769403f9bf271fd3573f86c.png)

## 指定存储后端consul

```bash
vim config.hcl
```

```bash
storage "consul" {
  address = "127.0.0.1:8500"
  path    = "vault/"
  token = "    "
}
```

```bash
vault server -dev -dev-listen-address=0.0.0.0:8200 -config=/root/config.hcl
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f889ca1465ee496494bbdac8f8bc2dca.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0bbd6b8f2841493982df458189cc3a86.png)

```bash
 curl  http://127.0.0.1:8500/v1/kv/vault/sys/expire/id/alicloud/creds/policy-based/iPed4e7I2dWZAIpN4UZkZDe9 --header "X-Consul-Token: 66911333-d553-f761-c425-a3aee5c0a165"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0cd81e8843e744beb87c2e3c70787e5d.png)

**常用接口**

```bash
curl   --header "X-Vault-Token: hvs.gbX5tfnfhlXxHskROzVCJdlZ"     http://127.0.0.1:8200/v1/alicloud/creds/policy-based

curl \
    --header "X-Vault-Token: hvs.CdprfudGwNYeTu2LX1wynrk2" \
    --request DELETE \
    http://127.0.0.1:8200/v1/alicloud/creds/policy-based/


curl     --header "X-Vault-Token: hvs.CdprfudGwNYeTu2LX1wynrk2"     --request POST        http://127.0.0.1:8200/v1/sys/leases/revoke/alicloud/creds/policy-based/B05ejqI42NnyFA4l6HHWi9Gz

```


```bash
curl \
    --header "X-Vault-Token: hvs.gbX5tfnfhlXxHskROzVCJdlZ" \
    --request LIST \
    http://127.0.0.1:8200/v1/auth/alicloud/roles
```

## consul

```bash
wget https://releases.hashicorp.com/consul/1.13.1/consul_1.13.1_linux_amd64.zip
unzip -d /usr/local/bin consul_1.13.1_linux_amd64.zip
consul agent -dev -ui -client 0.0.0.0
```
**CONSUL启用ACL**
```bash
mkdir /etc/consul.d
cd /etc/consul.d
vim consul-acl.hcl
```

```bash
acl = {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
}
```

```bash
consul agent -dev -ui -client 0.0.0.0 -config-dir=/etc/consul.d/
```

```bash
consul acl bootstrap
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f0b0e5328964ba2bc7b79089e2a1b54.png)

WEB UI输入SecretID

![在这里插入图片描述](https://img-blog.csdnimg.cn/000364424fcd49a7bdcce8db5982575c.png)

```bash
vault write consul/config/access  address="127.0.0.1:8500" token="eb88815c-2065-b1f8-7920-6ea7b5cb532a"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/402611ce89fb4ceab5f4f1b805ff779a.png)


