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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/61ac6ef1-eced-45f8-8363-c12fccc39f66)

```bash
vault write alicloud/config \
    access_key= \
    secret_key=
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/6774f5b9-4d2a-4ba8-b7b9-250373a14951)

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/a8e97583-8f94-4e10-b3ec-1294b30c8671)

**生成密钥**

在配置秘密引擎并且用户/机器拥有具有适当权限的 Vault 令牌后，它可以生成凭据。
/creds通过从具有角色名称的端点读取来生成新的访问密钥：

```bash
vault read alicloud/creds/policy-based
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/161bd42a-6069-4718-99d1-03f5f087d37b)

**回收密钥**

```bash
vault lease revoke alicloud/creds/policy-based/KSPlpAF9wwoc88Pg4wvVZyKg
```
可以看到生成了新的用户

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/04ad50af-d6c8-4f26-b59c-04b33d35fea6)

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/de4115a7-f80c-42ae-8ede-45039857a024)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/a97896d5-7e22-4989-bf23-941c7247bf14)

```bash
 curl  http://127.0.0.1:8500/v1/kv/vault/sys/expire/id/alicloud/creds/policy-based/iPed4e7I2dWZAIpN4UZkZDe9 --header "X-Consul-Token: 66911333-d553-f761-c425-a3aee5c0a165"
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/9821ec19-5d3a-4e96-9561-4c6b7209e383)

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

## consul秘钥

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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/642beb1c-eb9b-4cfa-9c54-ba1e969d5ced)

WEB UI输入SecretID

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/fcc4d31d-0ad3-482d-b135-f2ef5efec617)

```bash
vault write consul/config/access  address="127.0.0.1:8500" token="eb88815c-2065-b1f8-7920-6ea7b5cb532a"
```
![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/09548c3e-b060-42e0-ae26-a46bcda432b4)



