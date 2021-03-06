<!--
 * @Author: your name
 * @Date: 2020-02-15 10:42:18
 * @LastEditTime: 2020-04-24 22:43:27
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \git_study\linux_study\linux_user.md
 -->

# 控制用户(组)

## 控制用户

### 增加用户

1. 使用 useradd 增加用户
   * `useradd <用户名>` 这种方案仅仅为系统增加一个用户, 但不会为其在本地增加一个对应的用户名文件, 这样的话需要使用 `useradd -m <用户名>`来实现在本地增加用户文件
2. 使用 adduser 增加用户
   * `adduser <用户名>` 这种方案会直接生成一个用户及其文件
3. 使用 passwd 为一个用户设置密码(新增或者删减)
   * `passwd <用户名>` 激活变更密码的处理, 其中由于调用用户名
4. 位置
   * 用户: `/home/<username>`
   * 密码: `/etc/passwd/`

### 删除用户

`userdel <用户名>` 这样删除只会删除掉用户, 但本地的用户文件还在, 所以如果想要删的干净使用`userdel -r <用户名>`, 也即使用`-r`参数

### 切换用户

使用`su`(switch user)命令

## 修改账户信息

usermod (选项) (参数)

选项:

* -c<备注>：修改用户帐号的备注文字
* -d<登入目录>：修改用户登入时的目录
* -e<有效期限>：修改帐号的有效期限
* -f<缓冲天数>：修改在密码过期后多少天即关闭该帐号
* -g<群组>：修改用户所属的群组
* -G<群组>；修改用户所属的附加群组
* -l<帐号名称>：修改用户帐号名称
* -L<锁定账户>：锁定用户密码，使密码无效
* -s<设置shell>：修改用户登入后所使用的shell
* -u<用户ID>：修改用户ID
* -U:解除密码锁定

## 控制用户组

1. 增加: `groupadd <用户组名>`
2. 删除: `groupdel <用户组名>`
