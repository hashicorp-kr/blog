# Consul-TLS-configuration

## Certificate Creation Process for Consul

Just follow the guides below.

[Secure Agent Communication with TLS Encryption \| Consul - HashiCorp Learn](https://learn.hashicorp.com/consul/security-networking/certificates)

1. Creating Certificates
2. Configuring Agents
3. Configuring the Consul CLI for HTTPS
4. Configuring the Consul UI for HTTPS
5. Creating Certificates

   Consul의 TLS을 설정을 위한 certificates 생성. 동일한 CA가 서명한 인증서 필요. Private CA가 필요. 해당 작업은 Consul Server 노드에서 진행할 것.

6. Step 1. CA 생성 Consul이 설치되고 동작하는 데이터 센터를 위한 CA 생성

   ```text
   $ consul tls ca create
   ==> Saved consul-agent-ca.pem
   ==> Saved consul-agent-ca-key.pem
   ```

   CA Certificate : consul-agent-ca.pem → 모든 Consul Agent에 배포  
   CA Key : consul-agent-ca-key.pem

7. Step 2. 개별 서버 dc1 데이터센터의 consul 도메인을 위한 서버 인증서 생성. 개별 서버 개수만큼 실행.

   ```text
   $ consul tls cert create -server
   ==> WARNING: Server Certificates grants authority to become a
   server and access all state in the cluster including root keys
   and all ACL tokens. Do not distribute them to production hosts
   that are not server nodes. Store them as securely as CA keys.
   ==> Using consul-agent-ca.pem and consul-agent-ca-key.pem
   ==> Saved dc1-server-consul-0.pem
   ==> Saved dc1-server-consul-0-key.pem
   ```

   연속 수행 시 인증서와 키번호가 증가.

8. Step 3. Client 인증서 Consul 1.5.2부터 자동으로 인증서 배포 가능 \(auto\_encrypt\) 또는 수동 생성은 배포 가능 \($ consul tls cert create -client\)
9. Consul Agent 구성하기
10. Step 1. Consul 서버 설정하기

    ```yaml
    vagrant@consul:/vagrant/vault/cert$ cat /etc/consul.d/consul.hcl
    bootstrap_expect  = 1
    bind_addr         = "10.7.0.15"
    client_addr       = "0.0.0.0"
    data_dir          = "/var/lib/consul"
    datacenter        = "DC1"
    enable_syslog     = true
    log_level         = "INFO"
    server            = true
    encrypt           = "KnqifJT6qkF0X+Zx9spnjg=="
    ui                = true
    disable_host_node_id = true
    connect = {
      enabled = true
          }
    verify_incoming = true
    verify_outgoing = true
    verify_server_hostname = true
    auto_encrypt = {
      allow_tls = true
    }
    ca_file    = "/etc/ssl/consul-agent-ca.pem"
    cert_file = "/etc/ssl/dc1-server-consul-0.pem"
    key_file = "/etc/ssl/dc1-server-consul-0-key.pem"
    ports = {
     http = -1
     https = 8501
    }
    ```

11. Step 2. Consul Client 설정

    \`\`\`json

    vagrant@vault:/vagrant/vault/cert$ cat /etc/consul.d/consul.json

    {

      "advertise\_addr": "10.7.0.13",

      "client\_addr": "0.0.0.0",

      "data\_dir": "/var/lib/consul",

      "datacenter": "DC1",

      "enable\_syslog": true,

      "log\_level": "INFO",

      "ui\_dir": "/opt/consul-ui",

      "retry\_join" : \["10.7.0.15:8301"\],

      "encrypt": "KnqifJT6qkF0X+Zx9spnjg==",

      "connect": {

    ```text
    "enabled" : true
    ```

      },

      "ca\_file": "/etc/ssl/consul-agent-ca.pem",

      "cert\_file": "/etc/ssl/dc1-server-consul-0.pem",

      "key\_file": "/etc/ssl/dc1-server-consul-0-key.pem",

    "auto\_encrypt": {

      "tls": true

    },

    "ports": {

      "http": -1,

      "https": 8501

    }

    }

```text
3. Configuring the Consul CLI for HTTPS
```cli
$ consul tls cert create -cli
==> Using consul-agent-ca.pem and consul-agent-ca-key.pem
==> Saved dc1-cli-consul-0.pem
==> Saved dc1-cli-consul-0-key.pem
$ consul members -ca-file=consul-agent-ca.pem -client-cert=dc1-cli-consul-0.pem \
  -client-key=dc1-cli-consul-0-key.pem -http-addr="https://localhost:8501"
  Node     Address         Status  Type    Build     Protocol  DC   Segment
```

or

```text
$ export CONSUL_HTTP_ADDR=https://localhost:8501
$ export CONSUL_CACERT=consul-agent-ca.pem
$ export CONSUL_CLIENT_CERT=dc1-cli-consul-0.pem
$ export CONSUL_CLIENT_KEY=dc1-cli-consul-0-key.pem
```

## Vault Server Config with TLS Enabled Consul

```javascript
vagrant@vault:/vagrant/vault/cert$ cat /etc/vault.d/vault-config.json
{
  "backend": {
    "consul": {
      "scheme" : "https",
      "address": "127.0.0.1:8501", # This is important
      "path": "vaulttls/",
      "tls_ca_file" : "/etc/ssl/consul-agent-ca.pem",
      "tls_cert_file" : "/etc/ssl/dc1-server-consul-0.pem",
      "tls_key_file" : "/etc/ssl/dc1-server-consul-0-key.pem"
    }
  },
  "listener": {
    "tcp":{
      "address": "0.0.0.0:8200",
      "tls_disable": 1
    }
  },
  "ui": true,
  "disable_mlock": true,
  "log_level": "DEBUG"
}
```

1. Configuring the Consul UI for HTTPS
2. Step 1. Interface binding

   ```javascript
   {
   "ui": true,
   "client_addr": "0.0.0.0",
   "enable_script_checks": false,
   "disable_remote_exec": true
   }
   ```

3. Step 2. verify\_incoming\_rpc

   ```bash
   $ curl https://localhost:8501/ui/ -k -I
   curl: (35) error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate
   ```

   ```javascript
   {
   "verify_incoming": false,
   "verify_incoming_rpc": true
   }
   ```

   ```bash
   $ curl https://localhost:8501/ui/ -k -I
   HTTP/2 200
   ...
   ```

4. Step 3. Subject Alternative Name

   ```bash
   $ curl https://consul.example.com:8501/ui/ \
   --resolve 'consul.example.com:8501:127.0.0.1' \
   --cacert consul-agent-ca.pem
   curl: (51) SSL: no alternative certificate subject name matches target host name 'consul.example.com'
   ...
   ```

   ```bash
   $ consul tls cert create -server -additional-dnsname consul.example.com
   ...
   ```

   ```bash
   $ curl https://consul.example.com:8501/ui/ \
   --resolve 'consul.example.com:8501:127.0.0.1' \
   --cacert consul-agent-ca.pem -I
   HTTP/2 200
   ...
   ```

5. Step 4. Trust the Consul CA

   ```bash
   $ curl https://consul.example.com:8501/ui/ \
   --resolve 'consul.example.com:8501:127.0.0.1'
   curl: (60) SSL certificate problem: unable to get local issuer certificate
   ...
   ```

   You can fix that by trusting your Consul CA \(consul-agent-ca.pem\) on your machine, please use Google to find out how to do that on your OS.

