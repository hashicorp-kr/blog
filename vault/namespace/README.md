# NameSpace-만들기

```text

vagrant@vault:~$ vault namespace list
No namespaces found
vagrant@vault:~$ vault namespace create SE/
Key     Value
---     -----
id      3KmtI
path    SE/
vagrant@vault:~$ vault namespace create CL/
Key     Value
---     -----
id      veIPQ
path    CL/
vagrant@vault:~$ vault namespace create WL/
Key     Value
---     -----
id      FTr0q
path    WL/
vagrant@vault:~$ vault namespace list
Keys
----
CL/
SE/
WL/
vagrant@vault:~$ vault namespace lookup SE/
Key     Value
---     -----
id      3KmtI
path    SE/
vagrant@vault:~$ vault namespace create -namespace=SE/ T1
Key     Value
---     -----
id      el0Ts
path    SE/T1/
vagrant@vault:~$ vault namespace lookup -namespace=SE/ T1
Key     Value
---     -----
id      el0Ts
path    SE/T1/
vagrant@vault:~$ vault namespace list 
Keys
----
CL/
SE/
WL/
vagrant@vault:~$ vault namespace list -namespace=SE
Keys
----
T1/
```

