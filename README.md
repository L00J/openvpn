<!--
 * @Author: Logan.Li
 * @Gitee: https://gitee.com/attacker
 * @email: admin@attacker.club
 * @Date: 2025-04-27 12:15:39
 * @LastEditTime: 2025-04-27 12:16:41
 * @Description: 
-->
# openvpn

openvpn install

## 证书

```txt
ca.crt 根证书到期时间:  99年
server.crt  服务器签约证书到期时间:  99年
```

## 手动生成证书

```bash
cd /etc/openvpn
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
cp vars.example  vars
sed -i '/^#/d;/^$/d' vars # 清理注释

cat >> vars <<EOF
set_var EASYRSA_DN      "org"
set_var EASYRSA_REQ_COUNTRY     "CN"
set_var EASYRSA_REQ_PROVINCE    "Zhejiang"
set_var EASYRSA_REQ_CITY        "HangZhou"
set_var EASYRSA_REQ_ORG         "opsbase.cn"

set_var EASYRSA_REQ_EMAIL       "admin@attacker.club"
set_var EASYRSA_REQ_OU          "Organizational Unit"

set_var EASYRSA_CA_EXPIRE 36135
# 根证书99年
set_var EASYRSA_CERT_EXPIRE 3650
# 签约服务端证书10年
EOF

rm pki/ -rf # 清理历史pki目录
./easyrsa init-pki # 初始化pki目录
./easyrsa build-ca  nopass # 创建根证书
# /etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt
./easyrsa gen-req server nopass # 创建服务端证书
# req: /etc/openvpn/easy-rsa/easyrsa3/pki/reqs/server.req

# key: /etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key

./easyrsa sign server server # 服务器证书签约 (yes|确认)
#/etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt

./easyrsa gen-dh # 创建Diffie-Hellman
```

## 生成客户端证书（可设置99年有效期）

> 默认证书有效期为825天，如需99年（36135天），可加参数 `--days=36135`

### 1. 切换到 easy-rsa 目录
```bash
cd /etc/openvpn/easy-rsa/easyrsa3
```

### 2. 生成客户端私钥和请求（以 client 为例，99年有效期）
```bash
./easyrsa --days=36135 gen-req client nopass
```
- 生成私钥：`pki/private/client.key`
- 生成请求：`pki/reqs/client.req`

### 3. 用CA签发客户端证书（99年有效期）
```bash
./easyrsa --days=36135 sign client client
```
- 证书路径：`pki/issued/client.crt`
- 输入 `yes` 确认签发

### 4. 拷贝客户端所需文件
- `pki/ca.crt`                # CA根证书
- `pki/issued/client.crt`     # 客户端证书
- `pki/private/client.key`    # 客户端私钥

### 5. .ovpn 配置片段
```conf
client
dev tun
proto udp
remote <服务器公网IP> 1194
ca ca.crt
cert client.crt
key client.key
...
```

> 多个客户端证书只需更换 client 为不同名称重复上述步骤。
