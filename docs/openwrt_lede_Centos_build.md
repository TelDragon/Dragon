# 创建系统账户
useradd yuanyl
passwd yuanyl

# 编辑 visudo 以防出现 “s not in the sudoers file. This incident will be reported.”

root ALL=(ALL) ALL
yuanyl ALL=(ALL) ALL     //添加的配置

# 使用 yuanyl 账户登录

# 安装依赖软件
sudo yum install subversion binutils bzip2 gcc gcc-c++ gawk gettext flex ncurses-devel zlib-devel zlib-static make patch unzip perl-ExtUtils-MakeMaker glibc glibc-devel glibc-static quilt ncurses-libs sed sdcc intltool sharutils bison wget git-core openssl-devel xz

# 建立工作目录
pwd
/home/yuanyl
mkdir lede

# git 克隆 LEDE项目
git clone https://git.lede-project.org/source.git           //稳定17.1
git clone https://github.com/lede-project/source.git        //最新 17.4

# 然后将source目录更名为 source_master 注意: 编译将在此目录下执行, 根据官方文档的要求, 路径中要避免出现空格等特殊字符, 以免编译失败
mv source source_master

# 更新组件源
./scripts/feeds update -a

# 安装所有组件, 安装后, 在make menuconfig中才可以选择 如遇权限问题请加权限“chmod +x feeds”
./scripts/feeds install -a

# 安装指定组件
./scripts/feeds install <PACKAGENAME>
  
# 重置默认的设备和组件 如遇到权限被拒绝干脆“sudo chmod -R 777 source_master”
make defconfig
  
# 进入配置界面
make menuconfig

## 选择项目
  800MHz VIA Cortex-A9 SoC 属于 Broadcom BCM47xx/53xx (ARM) 系列
  Target System (Broadcom BCM47xx/53xx (ARM))  --->  
  Target Profile (Broadcom SoC, BCM43xx WiFi (b43, brcmfmac, default))  --->
  添加图形界面
  添加中文语言包
  添加界面主题
  
# 预先下载dl库，可以避免下载造成的编译失败。
make download V=s


  
# 开始编译
make FORCE_UNSAFE_CONFIGURE=1 -j4 V=s                          //root用户编译

make -j2 V=s TARGET_DEVICES=hc5761
# -j2 表示用2核, 根据当前机器配置而定
# V=s 表示显示详细输出

# 后台编译方法
nohup make -j2 V=s TARGET_DEVICES=hc5761 > ~/lede/logs/20171011.log 2>&1 &
  
  
常见问题

1. 编译时因网络问题下载失败

自行下载文件后放到dl目录下

2. 编译transmission时出现 config.status: error: cannot find input file: `po/Makefile.in.in‘ 错误

这是因为没安装glib2-devel
  
  http://koolshare.cn/thread-93843-1-1.html
  
3. 出现 error GNU libiconv not in use but included iconv.h is from libiconv
在glib目录下，手动执行：
./configure --enable-iconv=no --with-libiconv=gnu

  然后再编译
