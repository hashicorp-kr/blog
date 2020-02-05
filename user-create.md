# User-Create

## User Pass 활성화

```text

vagrant@vault:~$ vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
vagrant@vault:~$ vault auth list
Path         Type        Accessor                  Description
----         ----        --------                  -----------
token/       token       auth_token_14c42341       token based credentials
userpass/    userpass    auth_userpass_5ca567a5    n/a
vagrant@vault:~$ vault auth help userpass/
Usage: vault login -method=userpass [CONFIG K=V...]

  The userpass auth method allows users to authenticate using Vault's
  internal user database.

  If MFA is enabled, a "method" and/or "passcode" may be required depending on
  the MFA method. To check which MFA is in use, run:

      $ vault read auth//mfa_config

  Authenticate as "sally":

      $ vault login -method=userpass username=sally
      Password (will be hidden):

  Authenticate as "bob":

      $ vault login -method=userpass username=bob password=password

Configuration:

  method=
      MFA method.

  passcode=
      MFA OTP/passcode.

  password=
      Password to use for authentication. If not provided, the CLI will prompt
      for this on stdin.

  username=
      Username to use for authentication.
vagrant@vault:~$ 
```

### Namespace에 UserPass활성화 하기

```text

vagrant@vault:~$ vault auth enable -namespace="SE/" userpass
Success! Enabled userpass auth method at: userpass/
```

### NameSpace에 사용자 추가하기

```text

vagrant@vault:~$ vault write  -namespace=SE/ auth/userpass/users/u001 password="welcome1!"
Success! Data written to: auth/userpass/users/u001
vagrant@vault:~$ vault login -namespace=SE/  -method=userpass username=u001
Password (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.CYY8BEMiSx8sTp5KYMpmnFWU.3KmtI
token_accessor         6PbAyLpfWoqHu4vcXdIsxRTM.3KmtI
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    u001
vagrant@vault:~$ vault login s.CYY8BEMiSx8sTp5KYMpmnFWU.3KmtI
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.CYY8BEMiSx8sTp5KYMpmnFWU.3KmtI
token_accessor         6PbAyLpfWoqHu4vcXdIsxRTM.3KmtI
token_duration         767h58m59s
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    u001
```

## token 생성

```text

vagrant@vault:~$ vault token create -namespace=SE/
Key                  Value
---                  -----
token                s.PrFZz1DXeOEppA123dw4wvUK.3KmtI
token_accessor       JQvjfOcOFusifSWcRoF6IW1N.3KmtI
token_duration       768h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]
vagrant@vault:~$ vault token create -ns=SE/ 
Key                  Value
---                  -----
token                s.6zVh9LR4K0AmzZKlWCLZCVtO.3KmtI
token_accessor       Pul0Ri4zoRoTQDtFLdeWqgCl.3KmtI
token_duration       768h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]
```

