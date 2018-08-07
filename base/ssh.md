将公钥*id_rsa.pub*文件拷贝到服务器端的*~/.ssh/authorized_keys*文件中，有三种方法：

- 通过`scp`拷贝

  ```
  scp -P 22 ~/.ssh/id_rsa.pub user@host：~/authorized_keys
  ```

- 通过`ssh-copy-id`程序

  ```
  ssh-copy-id user@host
  ```

- 通过`cat`方法

  - Linux

    ```
    cat ~/.ssh/id_rsa.pub | ssh user@host "cat >> ~/.ssh/authorized_keys"
    ```

  - Windows

    ```
    type ~/.ssh/id_rsa.pub | ssh user@host "cat >> ~/.ssh/authorized_keys"
    ```