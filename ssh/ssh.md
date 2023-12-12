# Dùng ssh để remote vào command shell

## ssh vào command shell của server

```properties
$ ssh <user>@<ip-adress>
```

- Nếu dùng password authentication: nhập password khi được hỏi
- Nếu dùng pubkey authentication: nhập key vào file `.ssh/authorized_keys`

## Cấu hình file ssh

Cấu hình ssh nằm ở file `/etc/ssh/sshd_config`:

- Cho phép password authentication, bỏ comment:

```properties
#PasswordAuthentication yes
```

- Cho phép login bằng root user, bỏ comment:

```properties
#PermitRootLogin yes
```

- Cho phép key authentication, bỏ comment:

```properties
#PubkeyAuthentication yes
#AuthorizedKeysFile .ssh/authorized_keys
```

Sau khi thay đổi file cấu hình, restart service ssh bằng lệnh:

```properties
$ sudo systemctl restart ssh
```
