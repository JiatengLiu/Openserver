## 通过Jupyter下载文件夹

在 Jupyter 中，一般不能直接下载文件夹，需要先把文件夹压缩，然后再下载压缩包
1.  进入到文件夹所在目录下。
2.  在终端中执行压缩命令：
    ```bash
    zip -r 压缩包名称.zip 文件夹名称
    ```
    **例如：** 把名为 `original` 的文件夹压缩成 `copy.zip`
    ```bash
    zip -r copy.zip original
    ```
3.  在 Jupyter 文件列表中，选中新生成的压缩包下载即可。

---

## 通过 FileZilla 软件进行传输

1.  **下载软件**
    从官网下载 FileZilla Client：[Download FileZilla Client](https://filezilla-project.org/download.php)

2.  **新建站点**
    在软件左上角选择 **文件** → **站点管理器** → **新建站点**。
<img width="1183" height="937" alt="image" src="https://github.com/user-attachments/assets/df76c300-401d-4add-afa3-45f81884e2e4" />

3.  **配置 SFTP 连接**
   
    从您的服务器/容器实例找到 SSH 登录指令，并按如下方式填写信息。

    > **登录指令示例：** `ssh -p 12216 root@gh229.basic.cn`

    *   **协议(t):** `SFTP - SSH File Transfer Protocol`
    *   **主机(H):** `gh229.basic.cn` (指令中`@`后面的部分)
    *   **端口(P):** `12216` (指令中`-p`后面的数字)
    *   **用户(U):** `root` (指令中`@`前面的部分)
    *   **密码(W):** (填写您服务器的密码)


5.  **连接并传输**
    点击 **连接**，成功后即可在左右两个窗口间拖拽文件进行上传或下载。
