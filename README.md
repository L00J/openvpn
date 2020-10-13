# openvpn

openvpn install

## 证书

```txt
ca.crt 根证书到期时间:  99年
server.crt  服务器证书到期时间:  10年
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

