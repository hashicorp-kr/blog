# Key-Value-Secret-Engine

## Secret 만들기

```text

vagrant@vault:~$ vault kv put -ns=SE secret/hello foo=world
Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/internal/ui/mounts/secret/hello
Code: 403. Errors:

* preflight capability check returned 403, please ensure client's policies grant access to path "secret/hello/"
vagrant@vault:~$ vault secrets enable -ns=SE -path=secret/ kv
Success! Enabled the kv secrets engine at: secret/
vagrant@vault:~$ vault secrets list -ns=SE
Path          Type            Accessor                 Description
----          ----            --------                 -----------
aws/          aws             aws_55b4b02d             n/a
cubbyhole/    ns_cubbyhole    ns_cubbyhole_bfa7fa52    per-token private secret storage
identity/     ns_identity     ns_identity_d044f59a     identity store
secret/       kv              kv_e443e571              n/a
sys/          ns_system       ns_system_1f29e004       system endpoints used for control, policy and debugging
vagrant@vault:~$ vault kv put -ns=SE/ secret/hello foo=world
Success! Data written to: secret/hello
vagrant@vault:~$ vault kv put -ns=SE/ secret/hello foo=world excited=yes
Success! Data written to: secret/hello
```

## Secret 조회하기

```text

vagrant@vault:~$ vault kv get -ns=SE secret/hello
===== Data =====
Key        Value
---        -----
excited    yes
foo        world
vagrant@vault:~$ vault kv get -ns=SE -field=excited secret/hello
yes
vagrant@vault:~$ vault kv get -ns=SE -format=json secret/hello | jq -r .data.excited
yes
vagrant@vault:~$ vault kv get -ns=SE -format=json secret/hello
{
  "request_id": "73d39198-fb81-1310-7601-1d674e9d6bd6",
  "lease_id": "",
  "lease_duration": 2764800,
  "renewable": false,
  "data": {
    "excited": "yes",
    "foo": "world"
  },
  "warnings": null
}
```

## TTL 변경하기

```text

vagrant@vault:~$ vault kv get -ns=SE -format=yaml secret/hello
data:
  excited: "yes"
  foo: world
lease_duration: 2764800
lease_id: ""
renewable: false
request_id: b8937fc8-dbb2-daea-8379-a711fb9e8bf6
warnings: null
vagrant@vault:~$ vault secrets tune -ns=SE -default-lease-ttl=20m secret/
Success! Tuned the secrets engine at: secret/
vagrant@vault:~$ vault kv get -ns=SE -format=yaml secret/hello
data:
  excited: "yes"
  foo: world
lease_duration: 1200
lease_id: ""
renewable: false
request_id: 04e11e69-6a4a-ace8-dacf-4afa03e313f7
warnings: null
```

## Secret 삭제하기

```text

vagrant@vault:~$ vault kv delete -ns=SE secret/hello
Success! Data deleted (if it existed) at: secret/hello
```

