s3cmd 安装使用指南

https://wangyan.org/blog/s3cmd-how-to-use.html

s3cmd 安装使用指南

s3cmd 是一款 Amazon S3 命令行工具。它不仅能上传、下载、同步，还能设置权限，下面是完整的安装使用指南。

一、安装方法

方法一：（Debian/Ubuntu ）

  1 2 3	wget -O- -q http://s3tools.org/repo/deb-all/stable/s3tools.key \| sudo apt-key add - wget -O/etc/apt/sources.list.d/s3tools.list http://s3tools.org/repo/deb-all/stable/s3tools.list apt-get update && sudo apt-get install s3cmd
       	                                        

方法二：

  1 2 3 4	wget http://nchc.dl.sourceforge.net/project/s3tools/s3cmd/1.0.0/s3cmd-1.0.0.tar.gz tar -zxf s3cmd-1.0.0.tar.gz -C /usr/local/ mv /usr/local/s3cmd-1.0.0/ /usr/local/s3cmd/ ln -s /usr/local/s3cmd/s3cmd /usr/bin/s3cmd
         	                                        

二、使用方法

1、配置，主要是 Access Key ID 和 Secret Access Key

  1   	s3cmd --configure
      	                 

2、列举所有 Buckets。（bucket 相当于根文件夹）

  1   	s3cmd ls
      	        

3、创建 bucket，且 bucket 名称是唯一的，不能重复。

  1   	s3cmd mb s3://my-bucket-name
      	                            

4、删除空 bucket

  1   	s3cmd rb s3://my-bucket-name
      	                            

5、列举 Bucket 中的内容

  1   	s3cmd ls s3://my-bucket-name
      	                            

6、上传 file.txt 到某个 bucket，

  1   	s3cmd put file.txt s3://my-bucket-name/file.txt
      	                                        

7、上传并将权限设置为所有人可读

  1   	s3cmd put --acl-public file.txt s3://my-bucket-name/file.txt
      	                                        

8、批量上传文件

  1   	s3cmd put ./* s3://my-bucket-name/
      	                                  

9、下载文件

  1   	s3cmd get s3://my-bucket-name/file.txt file.txt
      	                                        

10、批量下载

  1   	s3cmd get s3://my-bucket-name/* ./
      	                                  

11、删除文件

  1   	s3cmd del s3://my-bucket-name/file.txt
      	                                      

12、来获得对应的bucket所占用的空间大小

  1   	s3cmd du -H s3://my-bucket-name
      	                               

三、目录处理规则

以下命令都能将dir1 中的文件上传至my-bucket-name，但效果只截然不同的。

1）dir1 不带"/"斜杠，那么dir1会作为文件路径的一部分，相当于上传整个dir1目录，即类似 "cp -r dir1/"

  1 2 	~/demo$ s3cmd put -r dir1 s3://my-bucket-name/ dir1/file1-1.txt -> s3://my-bucket-name/dir1/file1-1.txt  [1 of 1]
      	                                        

2）带"/"斜杠的 dir1，相当于上传dir1目录下的所有文件，即类似 "cp ./* "

  1 2 	~/demo$ s3cmd put -r dir1/ s3://my-bucket-name/ dir1/file1-1.txt -> s3://my-bucket-name/file1-1.txt  [1 of 1]
      	                                        

四、同步方法

这是s3cmd 使用难点，但却是最实用的功能。官方使用说明见《s3cmd sync HowTo》

首先明确，同步操作是要进行MD5校验的，只有当文件不同时，才会被传输。

4.1、常规同步操作

1、同步当前目录下所有文件

  1   	s3cmd sync  ./  s3://my-bucket-name/
      	                                    

2、加 "--dry-run"参数后，仅列出需要同步的项目，不实际进行同步。

  1   	s3cmd sync  --dry-run ./  s3://my-bucket-name/
      	                                        

3、加 " --delete-removed"参数后，会删除本地不存在的文件。

  1   	s3cmd sync  --delete-removed ./  s3://my-bucket-name/
      	                                        

4、加 " --skip-existing"参数后，不进行MD5校验，直接跳过本地已存在的文件。

  1   	s3cmd sync  --skip-existing ./  s3://my-bucket-name/
      	                                        

4.2、高级同步操作

4.2.1、排除、包含规则（--exclude 、--include）

file1-1.txt被排除，file2-2.txt同样是txt格式却能被包含。

  1 2 3	~/demo$ s3cmd sync --dry-run --exclude '*.txt' --include 'dir2/*' ./  s3://my-bucket-name/ exclude: dir1/file1-1.txt upload: ./dir2/file2-2.txt -> s3://my-bucket-name/dir2/file2-2.txt
       	                                        

4.2.2、从文件中载入排除或包含规则。(--exclude-from、--include-from)

  1   	s3cmd sync  --exclude-from pictures.exclude ./  s3://my-bucket-name/
      	                                        

pictures.exclude 文件内容

  1 2 3	# Hey, comments are allowed here ;-) *.jpg *.gif
       	                                        

4.2.3、排除或包含规则支持正则表达式

  1   	--rexclude 、--rinclude、--rexclude-from、--rinclude-from
