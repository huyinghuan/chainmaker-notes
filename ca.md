## chainmaker-ca 总结



### 编译部署

根据 [官方文档](https://docs.chainmaker.org.cn/operation/CA%E8%AF%81%E4%B9%A6%E6%9C%8D%E5%8A%A1.html)  git clone项目后，还需要执行

```
git submodule init && git submodule update
```

将子模块checkout出来然后编译出来。



### 配置文件

如果你是先部署的链服务，然后部署ca服务，那么配置文件中如下部分需要注意：

```yaml
# log config 省略，按需配置
log_config:
	...
# db config 省略，按需配置
db_config:
	...
# Base config
base_config:
  server_port: 9997                                # Server port configuration
  # 第一个注意点
  # 如果你是用官方的脚本 chainmaker-go/scripts/prepare_pwk.sh 启动的链
  # 那么可以选择 single_root 模式
  ca_type: single_root                               # Ca server type : double_root/single_root/tls/sign
  expire_year: 2                                     # The expiration time of the certificate (year)
  hash_type: SHA256                                  # SHA256/SHA3_256/SM3
  key_type: ECC_NISTP256                             # ECC_NISTP256/SM2
  can_issue_ca: false                                # Whether can continue to issue CA cert
  # 第二个主意的点
  # 这个地方不能错，错了点话，签署出来的证书服务被识别
  # 具体填的值，可以看到 chainmaker-go/build/crypto-config/ 下面的文件夹名称
  # 即几个组织id
  # 也可以通过长安链管理平台 的 区块链管理 -> 组织信息 -> 组织id列
  provide_service_for: [wx-org1.chainmaker.org]              # A list of organizations that provide services
  key_encrypt: false                                 # Whether the key is stored in encryption
  access_control: true                               # Whether to enable permission control

# Root CA config
root_config:
  cert:
    -
      cert_type: sign                                                  # Certificate path type : tls/sign (if ca_type is 'single_root',should be sign)
      # 第三个注意的点
      # 该证书应该使用链启动时点根证书
      # 即 chainmaker-go/build/crypto-config/xxx/ca下的证书，可拷贝他们到ca服务的目录下来
      cert_path: ./crypto-config/root/ca/ca.crt             # Certificate file path
      private_key_path: ./crypto-config/root/ca/ca.key      # private key file path
  # 第四个注意的点
  # 如果使用 链启动时的根证书，那么csr配置应该注释掉，否则会使用该配置生成的根证书用于 签署用户证书
  #csr:
  #  CN: root.org-wx                   
  #  O: org-wx                         
  #  OU: root                         
  #  country: CN                      
  #  locality: Beijing                
  #  province: Beijing             

# 第五个注意的点【官方文档有说明】
# 中间CA配置，默认启动时候，如果不注掉，他会在你配置的root级别上，再向下签发一级CA，所以你的链配置CA应该配置这个中间CA才能通过
# 如果使用 链启动时的根证书，那么注释以下配置即可
# intermediate_config: 
#  -
#    csr:
#      CN: imca                         
#      O: org1                        
#      OU: ca                         
#      country: CN                       
#      locality: Beijing                
#      province: Beijing            
#    private_key_pwd: wx1234

#access control config:
access_control_config:
  -
    app_role: admin
    app_id: admin
    app_key: passw0rd

```






