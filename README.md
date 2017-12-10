# Linux-Users-and-Groups
Các vấn đề liên quan đến permission. Một số ví dụ minh họa cách đặt và thay đổi permission cho user và group

## Nội dung 

[1. Permissions là gì ?](#permission)

[2. Làm việc với Users, Groups, và Directories](#user_group)

[3. Sudo](#sudo)

[4. Làm việc với Groups](#group)

[5. Tạo và xóa directory](#directory)

[6. Thay đổi permission của file và directory](#change_permission)

[7. Lệnh chmod](#chmod)

[8. Lệnh chown](#chown)

[9. Lệnh chgrp](#chgrp)

[10. Default Permissions](#umask)

[11. Tham khảo](#reference)

<a name="permission"></a>
## 1. Permissions là gì ?

Linux/UNIX là một hệ điều hành đa nhiệm (multitask) như các OS khác, sự khác biệt lớn nhất nằm ở khả năng đa người dùng (multiuser). Linux cho phép nhiều hơn một người có thể truy cập hệ thống trong một cùng thời gian. Để cho khả năng đa người dùng hoạt động hiệu quả, phải có cơ chế bảo vệ người dùng khỏi tác động ảnh hưởng lẫn nhau. Điều này làm nảy sinh khái niệm permission.

### 1.1. Read, Write & Execute Permissions

Permissions là cái "quyền" hoạt động trên file hoặc thư mục. Có 3 quyền cơ bản là đọc, ghi, thực thi: 

- Đọc: quyền có thể đọc cho phép nội dung file có thể xem. Áp dụng với thư mục thì bạn có thể liệt kê nội dung thư mục đó.

- Ghi: quyền ghi cho phép bạn có thể sửa đổi nội dung file đó. Áp dụng với thư mục thì bạn có thể chỉnh sửa nội dung thư mục đó (add/delete files)

- Thực thi: với file, quyền thực thi cho phép bạn chạy file và thực thi chương trình hoặc script. Với thư mục thì có thể cd vào trong nó. User sẽ thuộc group mặc định và có thể thuộc nhiều group khác.

### 1.2. Viewing File Permissions

Để xem các permission của file/dir ta dùng lệnh :

	$ ls -l 
	
Ví dụ 

	$ ls -l /etc/passwd 
	-rw-r--r-- 1 root root 2240 Feb 14 09:22 /etc/passwd
	
10 kí tự đầu tiên cho biết các quyền truy cập. Dấu trừ (-) đầu tiên cho biết loại file, d là directory, s là file đặc biệt, - là regular file. 3 kí tự tiếp là quyền cho người dùng đó, 3 ký tự tiếp nữa là cho nhóm người dùng, và 3 ký tự cuối cho tất cả những người dùng khác.

<a name="user_group"></a>
## 2. Làm việc với Users, Groups, và Directories 
	
### 2.1. Tạo và xóa user account

Sử dụng `useradd` để tạo một user theo cú pháp:

	# useradd <name>

Một vài option với chức năng đi kèm :

| Option | Mô tả | Cú pháp |
|--------|-------|---------|
| `-d <home_dir>` | Tạo ra thư mục home của user khi login | `useradd <name> -d /home/<user's home>` |
| `-e <date>` | Thời gian account bị mất hiệu lực | `user add <name>** -e <YYYY-MM-DD>` |
| `-f <inactive>` | Số ngày trước khi hết hiệu lực | `-f <inactive>` |
| `-s <shell>` | Cài đặt shell mặc định | `-s <shell>` |

Bạn cần đặt password cho user mới, sử dụng lệnh `passwd`. Lưu ý, bạn cần quyền root để thay đổi một password của người dùng nào đó.

	# passwd <username>
	
Người dùng cũng có thể thay đổi password của chính họ 
	
	$ passwd 
	Changing password for locvu.
	(current) UNIX password:
	Enter new UNIX password:
	Retype new UNIX password:
	passwd: password updated successfully

Có một cách tạo user account dễ dàng hơn, đó là sử dụng lệnh `adduser` (phân biệt với `useradd`). `adduser` sẽ tự động tạo thư mục home, group, shell mặc định.

	# adduser <name>
	
Nếu lệnh này chưa được OS hỗ trợ, có thể phải cài gói này bằng lệnh 

	# apt-get install adduser
	
Sau khi chạy lệnh `adduser`, bạn sẽ nhận được một loạt các yêu cầu nhập thông tin. Hầu hết đều là option có thể enter để bỏ qua, tuy nhiên bạn nên nhập Name và tất nhiên cả password nữa.

Để xóa một account, ta dùng lệnh 

	# userdel <name>

Sau khi thực hiện lệnh này, chỉ user account đó bị xóa, còn các file và thư mục home của user đó không bị xóa.

Để xóa user account, các file và thư mục home của chúng, ta thêm option `-r`

	# userdel -r <name>
	
<a name="sudo"></a>
## 3. Sudo

Root là super user, nó có khả năng làm mọi thứ trên hệ thống. Sudo cho phép các user và group sử dụng các lệnh mà thông thường không thể sử dụng được. Nó cho phép user có quyền như một admin mà không phải đăng nhập bằng tài khoản root. 

Ví dụ sử dụng sudo để cài một gói :

	$ sudo apt-get install <package>
	
Trước khi sử dụng `sudo` có thể bạn phải cài nó nếu distro đó không có :

Với các dòng Debian

	# apt-get install sudo 
	
hoặc với CentOS

	# yum install sudo
	
Để một user có thể dùng sudo, chúng cần được thêm vào file sudoer. File này rất quan trọng, không nên edit trực tiếp bằng editor.

Thay vì thế, ta sẽ sử dụng lệnh `visudo` với quyền root 

	# visudo 
	
Đây là phần cho biết các user có thể sử dụng sudo 

	# User privilege specification
	root    ALL=(ALL:ALL) ALL
	locvu  ALL=(ALL:ALL) ALL
	huyentrang  ALL=(ALL:ALL) ALL
	hexa ALL=(ALL:ALL) ALL
 
Bạn có thể chỉnh sửa tương tự theo mẫu đối với user cần sử dụng sudo.

Sau đó lưu file và đăng xuất khỏi root. Và đăng nhập với user vừa cấp quyền sudo và thực hiện chức năng tương tự root bằng cách thêm sudo ở đầu. 

Ví dụ : 

	$ sudo visudo

<a name="group"></a>
## 4. Làm việc với Groups

Linux sử dụng Group để tổ chức các User. Kiểm soát các thành viên trong group trong file `/etc/group`, trong đó sẽ có danh sách các group và thành viên của group đó. 

Ví dụ lệnh sau sẽ in ra file `/etc/group` và lọc lấy dòng thông tin về group sudo 

	cat /etc/group | grep sudo
	sudo:x:27:huyentrang,locvu,triduc

Một user có thể thuộc nhiều nhóm, tuy nhiên tại một thời điểm chỉ có một nhóm có hiệu lực. Để xem các nhóm mà user đó thuộc về ta sử dụng : 

	groups <username>
	
hoặc 

	id <username>

Mỗi user sẽ có một group mặc định, hay còn gọi là nhóm chính. Khi người dùng đăng nhập, user đó sẽ là thành viên của nhóm chính. Điều đó có nghĩa là, khi user đó khởi chạy một chương trình hoặc tạo file, chương trình và file đó sẽ liên kết với các thành viên trong group hiện tại của user đó.

Một user có thể truy cập vào một file thuộc nhóm khác nếu họ cũng là thành viên của nhóm đó và được cấp quyền truy cập. 

Để tạo file hay chạy một chương trình trong một group khác, ta phải chuyển đổi sang group đó bằng lệnh :   

	$ newgrp <group_name>

User cũng có thể thay đổi group cho file bằng lệnh :

    $ chgrp <group_name> <file_name>

<a name="directory"></a>
## 5. Tạo và xóa thư mục
	
Để tạo thư mục mới, sử dụng lệnh : 

	$ mkdir <directory name>
	
Vừa tạo thư mục mới, vừa thiết lập permission ta sử dụng thêm option `-m` như sau :

	$ mkdir -m a=rwx <directory name>
	
`-m` là viết tắt của mode, `a=rwx` là tất cả (all) các user đều có quyền read, write và execute trên thư mục này.

Để xóa file : 

	$ rm <file>
	
Để xóa thư mục : 

	$ rm -r <directory name>
	
Option `r` (recursive) là xóa toàn bộ thư mục, các thư mục con và nội dung của chúng. 

<a name="change_permission"></a>
## 6. Thay đổi permission của file và directory

Để thấy được các permission và quyền sở hữu của file và directory, ta sử dụng lệnh 

	$ ls -la

Option `a` (all) là hiển thị tất cả các file (bao gồm cả file ẩn)

Option `l` (long listing) là hiển thị mỗi file một dòng
	
	total 152
	drwx------ 5 locvu locvu   4096 Feb 15 18:38 .
	drwxr-xr-x 6 root  root    4096 Feb 15 14:10 ..
	-rw------- 1 locvu locvu   1030 Feb 10 09:29 .bash_history
	-rw-r--r-- 1 locvu locvu    220 Jan 17 15:42 .bash_logout
	-rw-r--r-- 1 locvu locvu   3637 Jan 17 15:42 .bashrc
	drwx------ 2 locvu locvu   4096 Jan 23 16:32 .cache
	-rw-rw-r-- 1 locvu locvu    195 Feb  7 10:36 check_dis.sh
	
Cột đầu là 10 ký tự về permission

Cột 2 là số file hoặc directory trong directory đó

Các cột tiếp đó là owner, group, size, date và time của lần truy cập gần nhất, tên file/dir

*Lưu ý :* Một directory có thể coi là một file, size của nó là 4096, nó không phải kích thước nội dung chứa trong directory đó.

<a name="chmod"></a>
## 7. Lệnh chmod 

### 7.1 Vài nét về `chmod`

Lệnh `chmod` viết tắt của change mode. Nó dùng để thay đổi permission của file/dir. Chúng ta có thể sử dụng chữ hoặc số (octal) để thiết lập permission. Với chữ, ta áp dụng theo bảng sau: 

| Kí tự | Permission | 
|-------|------------|
| r | Read |
| w | Write |
| x | Execute |
| X | Là x nhưng chỉ áp dụng với directory | 
| u | Người sở hữu (owner) |
| g | Nhóm sở hữu (group) |
| o | Các user khác (other)| 
| a | All, tương đương với u+g+o |

Ví dụ thêm (`+`) quyền execute cho owner, và bỏ (`-`) quyền read của group 

	$ chmod u+x,g-r <file-name>
 
### 7.2 Phân quyền theo dạng Octal 

Quy ước : 

	r = 4
	w = 2
	x = 1
	- = 0
	
Tính tổng 3 số của mỗi phần phân cho các đối tượng người dùng ta được một số đại diện cho phần đó

Ví dụ :

    rwx = 7
	rw- = 6
	r-x = 5
    r-- = 4
    ...

Ví dụ lệnh cấp quyền rwx cho owner, r-x cho group và other :

	$ chmod 755 <file-name>

	
### 7.3 Sticky bit và SUID/SGID

##### Sticky bit (octal 1000)

Bên cạnh +r, +w, +x còn có một số mode khác có thể hữu ích. Đặc biệt là +t (sticky bit), u+s (suid) và g+s (guid) 
	
Khi file/dir được đặt +t thì chỉ có owner và root mới có thể delete file. Kể cả những người có quyền write vào file/dir cũng không thể xóa chúng. 

Để thêm sticky bit ta theo lệnh sau : 

	$ chmod +t <file-name>
	
Kết quả 

	-rw-rw-r-T 1 locvu locvu    0 Feb 15 14:22 file-example


Với dir :
	
	$ chmod +t playground

Kết quả 
	
	drwxrwxrwt  2 locvu locvu    4096 Dec  9 20:58 playground

Cũng có thể thêm sticky bit theo hệ cơ số 8 

	$ chmod 1xxx <file-name>

Để xóa sticky bit, ta dùng `chmod -t`. 

##### SUID/SGID (octal 4000/2000)

Một file có như sau : 

```
-rwsrwsr-x  1 trang trang   17 Dec  9 21:37 s1.sh*
```

Chữ `s` đầu tiên biểu diễn cho setuid bit, chữ `s` sau biểu diễn setgid bit được bật

suid (u+s) khi thiết lập trên file cho phép người dùng với quyền thực thi có khả năng chạy file đó với quyền của owner.

sgid (g+s) tương tự suid nhưng là dành cho nhóm 

Ví dụ file `work` được sở hữu bởi `root` và group `marketing`. Các thành viên trong nhóm `marketing` có thể chạy file `work` nếu họ là `root`. 

Để đặt `+s` trên file `/usr/bin/work` ta dùng lệnh : 

	# chmod g+s /usr/bin/work

`+s` đối với directory thì có phần khác. Những file được tạo trong directory có `+s` đó sẽ nhận quyền của user và group của directory đó, chứ không phải người tạo ra file đó và nhóm mặc định của họ. 

Để setgid (group id) trên một directory, ta sử dụng theo lệnh : 

	# chmod g+s /var/doc-store/

Để setuid (user id) trên một directory, ta sử dụng theo lệnh : 

	# chmod u+s /var/doc-store/

<a name="chown"></a>	
## 8. Lệnh chown

Lệnh chown (change owner) để thay đổi quyền sở hữu của file. Mặc định quyền sở hữu của file thuộc về user tạo ra file đó và group mặc định của user đó. 

Để thay đổi quyền sở hữu của file, ta sử dụng theo lệnh :

    # chown user_name:group_name <file-name> 
	
Với directory ta cần sử dụng thêm option `-R`, ví dụ : 

	# chown -R lunglinh:root directory

Kết quả 

	drwxrwsr-x 2 lunglinh root    4096 Feb 15 21:41 directory/

<a name="chgrp"></a>
## 9. Lệnh chgrp

Lệnh chgrp (change group) để thay đổi quyền sở hữu của nhóm

Cú pháp :

	# chgrp  <group>  <filename/dirname>

<a name="umask"></a>
## 10. Default Permissions

Permisstion của file/dir được xác định bởi 2 thành phần là "bare permisstion" và "mask"

Bare permisstion là giá trị được thiết lập sẵn, không thể thay đổi : 666 với file và 777 với dir 

Mask được thiết lập bởi người dùng bằng lệnh umask 

Permisstion được tính bằng `Bare permisstion` AND `Mask`

Theo mặc định : 

User thường có umask 0002 

Do đó khi permisstion mặc định cho file là 0664 và dir là 0775

User root có umask 0022

Do đó khi permisstion mặc định cho file là 0644 và dir là 0755

Để thay đổi umask, thêm vào cuối file `~/.bashrc` để áp dụng cho riêng user, còn `/etc/profile` hoặc `/etc/bash.bashrc` để áp dụng cho tất cả các user.

Ví dụ: 

	umask 0111
	
NOTE: đường dẫn trên có thể khác nhau tùy distro, ở đây mình dùng với Ubuntu 16.04

Thay đổi trên `~/.bashrc` được ưu tiên hơn.

<a name="acls"></a>
## 11. ACLs

File và dir có những permission cho owner, group, other user, tuy nhiên những permission này có giới hạn. Ví dụ, những user khác nhau không thể có những permission khác nhau trên cùng một file/dir.

Do đó ACLs ra đời nhằm giải quyết bài toán trên. ACLs hỗ trợ các hệ thống file ReiserFS, Ext2, Ext3, JFS, XFS.



<a name="reference"></a>
## 12. Tham khảo 

[Linux Users and Groups](https://www.linode.com/docs/tools-reference/linux-users-and-groups)

[User Account and Group Management](http://www.cae.wisc.edu/cae-account-management/)

[Users and Groups Administration in Linux](http://www.debianadmin.com/users-and-groups-administration-in-linux.html)

[Online Chmod Calculator](http://www.onlineconversion.com/html_chmod_calculator.htm)
