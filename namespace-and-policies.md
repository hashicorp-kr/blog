# NameSpace-&-Policies

[원본](https://learn.hashicorp.com/vault/operations/namespaces)

![Example Corp](https://d33wubrfki0l68.cloudfront.net/729f799d8b01cc33b86066e02d841d5896c45533/fe6fb/static/img/vault-multi-tenant.png)

위 그림과 같은 조직 구조 상에서 특권을 가진 클러스터 수준의 관리자, 조직 수준 관리자 그리고 팀 수준 관리자를 지정하는 방법을 실습하도록 하겠습니다.

## Step 1: NameSpace 생성

상위 조직용 NameSpace 생성: education

```text

vagrant@vault:~$ vault namespace create education
Key     Value
---     -----
id      5v64d
path    education/
```

하위 조직용 namespace 생성 : training, certification

```text

vagrant@vault:~$ vault namespace create -namespace=education training
Key     Value
---     -----
id      Ae3J0
path    education/training/
vagrant@vault:~$ vault namespace create -namespace=education certification
Key     Value
---     -----
id      tlI1X
path    education/certification/
vagrant@vault:~$ vault namespace list
Keys
----
CL/
SE/
WL/
education/
vagrant@vault:~$ vault namespace list -ns=education/
Keys
----
certification/
training/
```

## Step 2: 정책 설정

운영을 책임지는 수퍼유저 생성 및 팀 수준 관리자 생성

### 수저 유저를 위한 정책

* 네임 스페이스 생성 및 관리
* 정책 생성 및 관리
* 비밀 엔진 활성화 및 관리
* 엔티티 및 그룹 생성 및 관리
* 토큰 관리

다음 내용을 edu-admin.hcl로 저장.

```text

# Manage namespaces
path "sys/namespaces/*" {
   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Manage policies
path "sys/policies/acl/*" {
   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List policies
path "sys/policies/acl" {
   capabilities = ["list"]
}

# Enable and manage secrets engines
path "sys/mounts/*" {
   capabilities = ["create", "read", "update", "delete", "list"]
}

# List available secret engines
path "sys/mounts" {
  capabilities = [ "read" ]
}

# Create and manage entities and groups
path "identity/*" {
   capabilities = ["create", "read", "update", "delete", "list"]
}

# Manage tokens
path "auth/token/*" {
   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
```

### 팀 수준 관리자 training admin 정책 설정

* 자식 이름 공간 만들기 및 관리
* 정책 생성 및 관리
* 비밀 엔진 활성화 및 관리

다음 내용을 training-admin.hcl로 저장

```text

# Manage namespaces
path "sys/namespaces/*" {
   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Manage policies
path "sys/policies/acl/*" {
   capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# List policies
path "sys/policies/acl" {
  capabilities = ["list"]
}

# Enable and manage secrets engines
path "sys/mounts/*" {
   capabilities = ["create", "read", "update", "delete", "list"]
}

# List available secret engines
path "sys/mounts" {
  capabilities = [ "read" ]
}
```

### 정책 배포하기

작업 시 해당 네임스페이스에 배포될 수 있도록 주의할 것. VAULT\_NAMESPACE 또는 -namespace 옵션\(-ns\) 사용.

```text

vagrant@vault:~$ vault policy write -ns=education edu-admin edu-admin.hcl
Success! Uploaded policy: edu-admin
vagrant@vault:~$ vault policy write -ns=education/training training-admin training-admin.hcl
Success! Uploaded policy: training-admin
```

## Step 3: 엔티티 및 그룹 설정

education 네임스페이스에 "Bob Smith"라는 edu-admin 정책에 연결된 엔티티를 생성. bob이라는 알리아스를 가진 사용자를 비밀번호를 사용하도록 생성. 기본적으로 "Bob Smith"는 education/training 네임스페이스에 대한 권한이 없으므로 "Training Admin"이라는 Group을 만들고 training-admin 정책을 연결한다. Bob을 training admin 그룹의 멤버로 추가하면 작업이 완료.

```text

vagrant@vault:~$ vault auth enable -ns=education userpass
Success! Enabled userpass auth method at: userpass/

```

