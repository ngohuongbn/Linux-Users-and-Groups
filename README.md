# Linux-Users-and-Groups
Các vấn đề liên quan đến permission. Một số ví dụ minh họa cách đặt và thay đổi permission cho user và group

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

Có một cách tạo user account dễ dàng hơn, đó là sử dụng lệnh `adduser` (phân biệt với `useradd`). `adduser` sẽ tự động thư mục home, group, shell mặc định.

	# adduser <name>
	
Nếu lệnh này chưa được OS hỗ trợ, có thể phải cài gói này bằng lệnh 

	# apt-get install adduser
	
Sau khi chạy lệnh `adduser`, bạn sẽ nhận được một loạt các yêu cầu nhập thông tin. Hầu hết đều là option có thể enter để bỏ qua, tuy nhiên bạn nên nhập Name và tất nhiên cả password nữa.

Để xóa một account, ta dùng lệnh 

	# userdel <name>

Sau khi thực hiện lệnh này, chỉ user account đó bị xóa, còn các file và thư mục home của user đó không bị xóa.

Để xóa user account, các file và thư mục home của chúng, ta thêm option `-r`

	# userdel -r <name>
	
## 3. Hiểu về Sudo

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
	
## 4. Tạo và xóa thư mục
	
	