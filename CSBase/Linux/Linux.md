

# 文件相关命令
1. 新建文件夹
    ```shell
    mkdir `name`;
    ```
2. 删除文件夹
   ```shell
   rm -rf 'filename'
   ```
3. 删除文件
   1. 单个删除
        ```shell
        rm -f 'filename'
        ```
   2. 批量删除包含关键字的文件
        ```shell
        rm -f '*keyword*'
        ```
4. 查看文件
   - `vi [filename]`
   - `vim [filename]`
   - `tail -f [filename]`


# 进程相关命令
1. 查看进程
    ```shell
    ps - ef | grep redis # 查看redis进程
    ```
2. 查看端口占用情况
    ```shell
    netstat -nap | grep [端口号]
    ```

# 执行命令
1. 执行jar包
    - `nohup java -jar filename.jar`。执行jar并把输出追加到`nohup.out`
    - `java -jar xxx.jar &`。后台运行java程序，执行ctrl+c不会退出程序。


# 主机虚拟机交互命令
1. 从主机上传文件到linux:`rz`
2. 从linux下载文件到主机:`sz [filename]`

# 路径相关命令
1. 查看路径变量：`echo $PATH`