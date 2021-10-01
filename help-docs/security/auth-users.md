users:
- two types of users
    - human
    - bots
    
- kube can't manage users itself, need another plugin to manage users
- it can however manager service accounts for bots
- kube apiserver authenticates user/requests


auth methods:
- Static password file:
    - we can create a csv file with user, pw and uid in this format
    ```
    user1,pw1,uid1
    user2,pw2,uid2
    ```
    - then we can pass this as cli arg during kube-api-server process starting using flag `--basic-auth-file=/path/to/file.csv`
    - We can also add a 4th column for groups and add users to group
    - http request can be made to api server using creds `-u usr1:pw1`

- Static Token file:
    - we can create a static token file similar to user/pw and pass to kube-apiserver process using flag `--token-auth-file=/path/to/file.csv`
    ```
    token1,usr1,uid1,gr1
    token2,usr2,uid2,gr2
    ```
    - http request can be made to api server using token `--header "Authorization: Bearer <token"`
- Certificates:



- Identity services like kerberos etc.
