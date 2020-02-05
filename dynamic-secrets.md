# Dynamic-Secrets

* kv와 다르게 최초 접속 시 secret이 생성. 여기서는 AWS로 테스트. DB Create도 적용 가능.

## AWS Secret Engine 활성화

```text

vagrant@vault:~$ vault secrets enable -path=aws -namespace=SE/ aws
Success! Enabled the aws secrets engine at: aws/
```

## AWS Secret Engine 구성

```text

vagrant@vault:~$ vault write -namespace=SE/ aws/config/root \
> access_key=AAVCKEKEEMEMEME \
> secret_key=xxxerrrrrrrllkjklsdfljksfdfsdjlk \
> region=ap-northeast-2
Success! Data written to: aws/config/root
```

## Role 생성하기

```text

vagrant@vault:~$ vault write -namespace=SE/ aws/roles/test-role \
> credential_type=iam_user \
> policy_document=-< {
>   "Version": "2012-10-17",
>   "Statement": [
>     {
>       "Sid": "Stmt1426528957000",
>       "Effect": "Allow",
>       "Action": [
>         "ec2:*"
>       ],
>       "Resource": [
>         "*"
>       ]
>     }
>   ]
> }
> EOF
Success! Data written to: aws/roles/test-role
```

## Secret 생성하기

```text

vagrant@vault:~$ vault read -ns=SE aws/creds/test-role
Key                Value
---                -----
lease_id           aws/creds/test-role/5RwIYlNkshsxnr5O6UJNcl2f.3KmtI
lease_duration     768h
lease_renewable    true
access_key         AKIA266GU7ZPB45BKCME
secret_key         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
security_token     

vagrant@vault:~$ vault secrets tune -ns=SE -default-lease-ttl=30m -max-lease-ttl=30m aws
Success! Tuned the secrets engine at: aws/
vagrant@vault:~$ vault read -ns=SE aws/creds/test-role
WARNING! The following warnings were returned from Vault:

  * TTL of "768h0m0s" exceeded the effective max_ttl of "30m0s"; TTL value is
  capped accordingly

Key                Value
---                -----
lease_id           aws/creds/test-role/gfz63NIuX3rouhEVUZZ2UdNC.3KmtI
lease_duration     30m
lease_renewable    true
access_key         AKIA266GU7ZPD77YHQ5A
secret_key         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
security_token     
vagrant@vault:~$ vault secrets tune -ns=SE -default-lease-ttl=24h -max-lease-ttl=24h aws
Success! Tuned the secrets engine at: aws/
vagrant@vault:~$ vault read -ns=SE aws/creds/test-role
WARNING! The following warnings were returned from Vault:

  * TTL of "768h0m0s" exceeded the effective max_ttl of "24h0m0s"; TTL value
  is capped accordingly

Key                Value
---                -----
lease_id           aws/creds/test-role/lnkGi5SwWm750lMzdO0GHLYe.3KmtI
lease_duration     24h
lease_renewable    true
access_key         AKIA266GU7ZPHWTSDBHO
secret_key         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
security_token     
```

## Secret Revoke

```text

vagrant@vault:~$ vault lease revoke aws/creds/test-role/lnkGi5SwWm750lMzdO0GHLYe.3KmtI
All revocation operations queued successfully!
```

