# HW1.1学习笔记  
> 日期：2026.7.3

## 虚拟机安装心得  
1. 不要直接把软件下载到根目录上，要新建一个文件夹，把全部文件下载在文件夹里，避免文件散落在根目录中，导致系统要求提供某个文件时找不到  
2. 在VMware下载完成之后，新建虚拟机之前，记得检查好VMware虚拟网卡是否连接正常  
（可以按下win + R打开运行框，输入命令：ncpa.cpl，回车后就会弹出网络适配器窗口）  
3. 下载.iso映像文件之后，Windows系统可能直接挂载为虚拟光驱，此时如果想要移动映像文件的位置，要先弹出它  
（打开此电脑，找到多出的光盘图标，右键点击“弹出”）
4. __虚拟机可能会出现卡顿的情况，此时可以考虑：__
   * 关闭3D加速（虚拟机->显示器->取消勾选“加速3D图形”）
   * 检查虚拟机内存挂载[（Linux命令）](HW1.md:78)  
   * 检查虚拟机磁盘是否爆满[（Linux命令）](HW1.md:77) 
   * 清理各类缓存、临时文件、旧内核[（Linux命令）](HW1.md:79)
   * 更改磁盘大小或内存大小
     * 磁盘：
       1. 扩容：虚拟机->硬盘（SCSI）->扩展  
       2. 分区：（Linux命令：sudo growpart /dev/sda 3)
     * 内存：虚拟机->内存
  

## Linux常用命令总结
* ### 常用短词命令：
    * ___sudo：临时以管理员身份执行命令___
    * ___cd：切换目录___
        1. cd~：回家目录 
        2. cd..：上级目录 
        3. cd-：上一次目录
    * ls：列出目录内容
    * ___mkdir：创建文件夹___
    * touch：创建空文件
    * ___rmdir：删除空文件夹___
    * ___rm：删除文件/文件夹___
        1. rm file：删除文件
        2. rm -r dir：删除文件夹
        3. rm -rf：递归删除
    * cat：一次性读取文件全部内容
    * grep：文本检索过滤
        1. grep “关键词” file：检索文件文本
        2. grep -r “关键词”：递归搜索
    * kill：结束进程
    * ___df -h：磁盘分区使用情况（df -h /：根分区）___
    * ___free -h：内存使用___
    * ping IP：测试连通性
    * find：全盘搜索文件
    * ___输出重定向：>：覆盖 >>：追加___
* ### 命令行选项：
    * ___-r：递归（--recursive）___
    * ___-y：自动确认（--yes）___
    * -f：强制（--force）
    * ___-h：人性化单位（--human）___
    * -l：详细列表
    * -i：
        * 交互确认（--interactive，常与rm搭配）
        * 忽略大小写匹配（--ignore-case，常与grep搭配）
        * 直接修改源文件而不输出终端：（--in-place，常与sed搭配）
        * 打印文件inode号（--inode，常与ls，df搭配）
    * -v：打印详细执行过程（--verbose）
    * -n：模拟执行（--dry-run）
* ### 常用命令句子：
    * #### 关机，休眠与重启：
        1. ___shutdown -r now（标准关机）___
        2. ___sudo reboot / sudo reboot -f（强制重启）___
        3. sudo systemctl poweroff（关机）
        4. sudo systemctl reboot（重启）
        5. sudo systemctl suspend（休眠）
    * #### apt类命令：
        * ___更新软件清单缓存：sudo apt update___（__很常用，在安装任何新软件前、执行系统升级前、换源前必须写，注意它不下载任何新软件和文件__）
        * 更新系统和已安装软件：sudo apt upgrade（第一次装机会用）
        * ___安装软件：sudo apt install xxx -y___
        * ___卸载软件：sudo apt remove xxx -y___
        * ___一并卸载孤立无用的依赖：sudo apt autoremove -y（建议搭配sudo apt remove xxx使用）___
    * #### systemctl系统类命令：
        * __开机自启：sudo systemctl enable gdm3(立即启用：sudo systemctl enable --now gdm3)__
        * 关闭开机自启：把上一条命令的enable改成disable
        * 重启gdm3服务：sudo systemctl restart gdm3
    * #### 检查内存、磁盘空间等命令：
        * ___查看根分区磁盘空间：df -h /___
        * ___查看总内存空间：free -h___
    * #### 清理内存、磁盘空间命令：
        * ##### 清理缓存：
            * 清理页缓存： sudo sync; sudo sysctl -w vm.drop_caches=1
            * 全部清理：sudo sync; sudo sysctl -w vm.drop_caches=3
            * 一键清理简写：sudo sync && sudo sysctl vm.drop_caches=3
            * ___清理下载过的安装包缓存：sudo apt clean -y___
            * ___删除老旧无用包缓存：sudo apt autoclean -y___
            * 通用用户缓存：sudo rm -rf ~/.cache/*
            * Conda清理缓存：
                * ___一键全清：conda clean --all -y___
                * ___只删除无用解压包：conda clean -p -y___
                * 预览释放空间： conda clean -a -d
        * ##### 清理无用文件：
            * ___清空临时文件：sudo rm -rf /tmp/\* 或：sudo rm -rf /var/tmp/\* （一定要记得加\*号！）___
            * ___清除日志文件：sudo journalctl --vacuum-size=100M___
        * ##### 清理旧内核
            * 一键自动清理：___sudo apt autoremove --purge -y___
            * 一键批量清理 __所有__ 非当前内核：  
            sudo apt purge \$(dpkg -l 'linux-{image,headers}-*' | awk '/^ii/{print \$2}' | grep -v $(uname -r)) -y  
            sudo update-grub  
            ___属于激进清理，要确保当前内核已经正常运作（比如第一次装机）___
        
## Conda隔离原理个人理解  
* Conda的功能：实现多个Python版本自由切换而无需修改系统版本；只要更改环境，不同项目的包版本不会产生冲突
* Conda隔离的底层原理：
    1. 目录隔离：Conda的根目录下设置不同分区表示不同的隔离环境（Base，env1，env2，...），不同分区是独立的文件夹，它们都有自己独立的一套Python解释器、库目录等，保证了切换环境之后它们有专属各自的一套工具可用
    2. 运行时隔离（劫持环境变量）：
       * 当用户切换至某个环境时，Conda会修改Shell的环境变量，重排PATH，即：把该环境的/bin目录等插到PATH最前面，让当前环境运行pip等时系统优先匹配当前目录下的二进制库
       * 再利用关键的隔离变量覆盖：
            * PYTHONHOME、PYTHONPATH：锁定当前目录，禁止读取其他环境的文件、库目录
            * CONDA_PREFIX：标记当前环境路径
            * LD_LIBRARY_PATH：优先加载当前环境动态库
    3. Python解释器隔离：
        * 每个环境自带独立的Python解释器，限定在当前环境的目录
        * 不同版本的Python是独立二进制，底层C API隔离，版本间不交叉调用

