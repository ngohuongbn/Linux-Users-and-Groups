Để hiểu về ACLs, mình xem một ví dụ https://www.youtube.com/watch?v=6piQXXHTmqk

Mình sẽ giải thích ví dụ bằng text
 
Tạo 2 group : `teachers` và `students` cùng các user tương ứng `teacher1` thuộc group `teachers` và `student1` thuộc `students`

```
[root@centos ~]# groupadd teachers
[root@centos ~]# groupadd students
[root@centos ~]# useradd -Gteachers teacher1
[root@centos ~]# useradd -Gstudents student1
[root@centos ~]# passwd teacher1
Changing password for user teacher1.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@centos ~]# passwd student1
Changing password for user student1.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Tạo 2 dir foo/bar và 2 file public.txt và secret.txt. Nội dung 2 file tùy ý

```
[locvu@centos]$ cd /srv/
[locvu@centos srv]$ sudo mkdir -p foo/bar
[sudo] password for locvu:
[locvu@centos srv]$ sudo vi foo/public.txt
[locvu@centos srv]$ sudo vi foo/bar/secret.txt
```

Permission của các file/dir được tạo theo mặc định 

```
[locvu@centos srv]$ ls -l foo/
total 4
drwxr-xr-x. 2 root root 24 Dec 10 07:43 bar
-rw-r--r--. 1 root root 27 Dec 10 07:42 public.txt
[locvu@centos srv]$ ls -lR foo/
foo/:
total 4
drwxr-xr-x. 2 root root 24 Dec 10 07:43 bar
-rw-r--r--. 1 root root 27 Dec 10 07:42 public.txt

foo/bar:
total 4
-rw-r--r--. 1 root root 53 Dec 10 07:43 secret.txt
```

Thay đổi permision thành 700 cho tất cả các file 

```
[locvu@centos srv]$ sudo chmod -R 700 foo/
[locvu@centos srv]$ sudo ls -lR foo/
foo/:
total 4
drwx------. 2 root root 24 Dec 10 07:43 bar
-rwx------. 1 root root 27 Dec 10 07:42 public.txt

foo/bar:
total 4
-rwx------. 1 root root 53 Dec 10 07:43 secret.txt
```

Thiết lập ACLs cho các file/dir bằng lệnh `setfacl`

Các option : `-R` - áp dụng đệ quy cho các subdir, `-m` - modify ACLs, `-d` - áp dụng cho ACLs mặc định 

```
[locvu@centos srv]$ sudo setfacl -Rdm g:teachers:rwx foo/
[locvu@centos srv]$ sudo setfacl -Rm g:teachers:rwx foo/
[locvu@centos srv]$ sudo setfacl -dm g:students:rwx foo/
[locvu@centos srv]$ sudo setfacl -m g:students:rwx foo/
[locvu@centos srv]$ sudo setfacl -m g:students:rwx foo/public.txt
```

User `student1` xem được file `public.txt` nhưng không có quyền truy cập `bar`

```
[locvu@centos srv]$ su student1
Password:
[student1@centos srv]$ ll
total 0
drwxrwx---+ 3 root root 35 Dec 10 07:42 foo
[student1@centos srv]$ cd foo/
[student1@centos foo]$ ll
total 4
drwxrwx---+ 2 root root 24 Dec 10 07:43 bar
-rwxrwx---+ 1 root root 27 Dec 10 07:42 public.txt
[student1@centos foo]$ cat public.txt
this is a public text file
[student1@centos foo]$ cd bar/
bash: cd: bar/: Permission denied
```

Xem thông tin ACLs `bar` bằng lệnh `getfacl`

```
[student1@centos foo]$ getfacl bar/
# file: bar/
# owner: root
# group: root
user::rwx
group::---
group:teachers:rwx
mask::rwx
other::---
default:user::rwx
default:group::---
default:group:teachers:rwx
default:mask::rwx
default:other::---
```

User `teacher1` có thể truy cập vào được cả `public.txt` và `secret.txt`

```
[student1@centos foo]$ exit
exit
[locvu@centos srv]$ su teacher1
Password:
[teacher1@centos srv]$ cd foo/
[teacher1@centos foo]$ ls -lR
.:
total 4
drwxrwx---+ 2 root root 24 Dec 10 07:43 bar
-rwxrwx---+ 1 root root 27 Dec 10 07:42 public.txt

./bar:
total 4
-rwxrwx---+ 1 root root 53 Dec 10 07:43 secret.txt
[teacher1@centos foo]$ cat bar/secret.txt
this is a secret text file, only teachers know about
```

Xem thông tin ACLs của `foo`

```
[teacher1@centos srv]$ getfacl foo/
# file: foo/
# owner: root
# group: root
user::rwx
group::---
group:teachers:rwx
group:students:rwx
mask::rwx
other::---
default:user::rwx
default:group::---
default:group:teachers:rwx
default:group:students:rwx
default:mask::rwx
default:other::---
```





