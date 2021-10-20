# Linux下的IDEA

## 一、下载

1、首先下载`tar.gz`文件

2、验证文件校验和，以确保下载是否正确，文件是否损坏。

​	终端键入

```markdown
cd /home/edric97/Downloads

sha256sum ideaIU-2021.2.3.tar.gz
```

​	然后会弹出一串序列，检查是否与下载页面的 SHA-256校验和 的链接内容是否一样

3、键入

```markdown
tar -zxvf ideaIU-2021.2.3.tar.gz
```

4、键入

```markdown
cd idea-IU-212.5457.46
sudo chmod a=+rx bin/idea.sh
bin/idea.sh（这个也是以后打开IDEA的方式）
```

注意`idea-IU-212.5457.46`是下载`tar.gz`文件边上经过前三步操作生成的文件名

5、然后就是图形用户界面了，自己操作即可

## 二、日常打开IDEA

我自己的：

```markdown
cd /home/edric97/Downloads/idea-IU-212.5457.46

bin/idea.sh
```

