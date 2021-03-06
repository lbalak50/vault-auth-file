[![Go Report Card](https://goreportcard.com/badge/github.com/gites/vault-auth-file)](https://goreportcard.com/report/github.com/gites/vault-auth-file) [![Build Status](https://travis-ci.org/gites/vault-auth-file.svg?branch=master)](https://travis-ci.org/gites/vault-auth-file) [![codecov](https://codecov.io/gh/gites/vault-auth-file/branch/master/graph/badge.svg)](https://codecov.io/gh/gites/vault-auth-file)

# Vault Auth File
HashiCorp Vault authentication plugin for authenticating via Unix password like file.

# Authentication workflow

1. User name and password are sent to Vault.
2. Vault calculate password hash and compare it to hash stored in password file.
3. If succesfull, policies listed in password file are applayed to user.
4. Valid token is returned to user.

# Instalation

1. Set `plugin_directory` variable in Vault config.
    ```bash
    plugin_directory="/opt/vault/plugins"
    ```
2. Copy  `vault-auth-file` binary to plugin_directory.
3. Enable plugin in Vault.
    ```bash
    SHA_256SUM=`sha256sum /opt/vault/plugins/vault-auth-file/vault-auth-file|cut -d' ' -f1`
    vault write sys/plugins/catalog/vault-auth-file sha_256=$SHA_256SUM command=vault-auth-file
    vault auth-enable -path=file -plugin-name vault-auth-file plugin
    ```
# Configuration
Configuration endpoint is located at `auth/file/config`.
Configuration options:
* **path** (mandatory)  - path to password file (example: /opt/vault/etc/password-file)
* **ttl** (optional) - token TTL  (example: 1h)
* **max_ttl** (optional) - max token TTL (example: 2h)

Example:
```bash
vault write auth/file/config path=/opt/vault/etc/password-file ttl=1h max_ttl=2h
```
# Password file format
Password file format is similar to Unix/Linux password file:
```
username:password_hash:coma,separated,policy,list
```

**For now only SHA-512 hashes are supported.**

Example:
```
wac:$6$.R4zGSdU$UQbNz4pV/AuDxD0Su6qfeVRaKz6gsq3w7zD8ywhFFpF7vbtiBxEFq49SbNI8kNGPmZyMzJIelUFvf12tUknjE0:ops,dev
wacek:$6$AwBd/60MqRG8M1V2$mXPJ39lAs26otEjY4YvObn7lEN2UeZgsEE6ueeN0zWS96QBJQuJLUhLmf1LuvCk7.MYpNik7tl5CEdqr.3Is80:ops,dev,netops
zenek:$6$jG6ZxCOkrXI$r4za9aGwb/VVw3nB3vRyvO2njCzgyKKCPxMn.GOYkW0/WaEMQENpbEufrX6CAQqlsIDr0x9DUsAhIS8bL3OGf1:ops,dev,netops
gites:$6$spfjUPN4$6ap3h.6Fac23HO/CFTZpQYdwvZ8zFflZkCQMWVO.13pCFEOjw8sjVljiIU6SgAhRDwwUBK1DYvHmBdoz/3wef0:ops
gites2:$6$EBzUEPlL$sLnPV5wKqvWloHNf7rfaO2bG1wxGl7zda6Jy/qU3ChLuIlK2EujMIaIdJfHhwbCst60IHqkFAiZXMVhFTQx3b1:ops
```

# Login 
Login endpoint is located at `auth/file/login`.
Login options:
* **username** (mandatory) - username to login
* **password** (mandatory) - password for that user

Example:
```bash
vault write auth/file/login username=wac password=lubieplacki      
Key                     Value                         
---                     -----                         
token                   ffff3192-87cf-c3c9-e3af-1a3373fb6017                                                 
token_accessor          9689189d-548c-3c06-6c81-fb621f9e404c                                                 
token_duration          1h0m0s                        
token_renewable         true                          
token_policies          [default dev ops]             
token_meta_username     "wac"                         
token_meta_woop         "woop.sh" 
```
Also `auth/file/login/<username>` endpoint can be used. This make vault-auth-file plugin compatible with Vault client binary and userpass auth method.

Example:
```bash
vault write auth/file/login/wac username=wac password=lubieplacki 
Key                    Value                        
---                    -----                        
token                  1c24c452-5599-f308-1f2b-4dc3f1621584                                              
token_accessor         c7a57130-90a7-7bbc-279d-b984053b7ee3                                              
token_duration         768h                         
token_renewable        true                         
token_policies         [default dev ops]            
token_meta_username    wac                          
token_meta_woop        woop.sh                      
```
Example using userpass auth method:
```bash
vault login -method=userpass -path=file username=wac 
Password (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  08c7483b-3f8f-37fe-c648-2f6d019d0742
token_accessor         9af3c49e-5ac2-f062-4e17-6c0058a2cc20
token_duration         768h
token_renewable        true
token_policies         [default dev ops]
token_meta_username    wac
token_meta_woop        woop.sh
```
