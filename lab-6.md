# Lab-06


## Requirement

請設計一個 Shell Script 用來 依據我們提供的 user config 批次新增使用 。


## Step

- command
```bash=
vim addallusers.sh
chmod +x addallusers.sh

mv addallusers.sh /usr/local/bin

```


- addallusers.sh
```bash=
#!/bin/bash

filename=$1
while read line; do
        n_line=`echo $line | sed "s/,/ /g"`
        #echo $n_line
        read -r -a my_arr <<< $n_line;

        user="${my_arr[0]}";
        password="${my_arr[1]}";

        #sudo useradd -m ${user} -p ${password};
        sudo useradd -p $(openssl passwd -crypt ${password}) -m ${user}
        if [ $?=9 ]; then
                sudo userdel ${user};
                sudo useradd -p $(openssl passwd -crypt ${password}) -m ${user};
        fi
        unset my_arr[0];
        unset my_arr[1];
        for i in "${my_arr[@]}"; do
                sudo usermod -aG ${i} ${user}
        done
done < $filename
```


