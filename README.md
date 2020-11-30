# Wordpress Playbook

Playbook này sẽ cài đặt wordpress với Mariadb, PHP và máy chủ web Apache (LAMP)

Playbook sẽ thực hiện các tác vụ trên máy chủ được quản lý như: 

- Cài đặt PHP
- Cài đặt apache
- Cài đặt Mariadb
- Tạo các user và database cho wordpress trong mariadb
- Tải xuống và giải nén wordpress
- Thiết lập file `wp-config.php` để kết nối tới database. 
- Thiết lập quyền sở hữu và quyền thực thi cho thư mục root của web.

## Requirements

Playbook sử dụng để cài wordpress trên Ubuntu 18.04 và CentOS 7

## Roles

| Role | Description | var path | var name | 
|-------|------------| -------- | -------- |
| php_config | Cài đặt PHP và các phần mở rộng PHP cần thiết mà Wordpress cần | defaults/main.yml | php_ubuntu_packages<br>php_centos7_packages | 
| apache_install | Cài đặt apache2 làm web server | defaults/main.yml | apache_packages<br>apache_name_service<br>httpd_packages<br>httpd_name_service |
| mysql_install | Cài đặt `mariadb` và gói python cho mysql | defaults/main.yml | mysql_python_package_debian<br>mysql_python_package_rhel<br>mariadb_packages_ubuntu<br>mariadb_packages_centos<br>sql_name_service |
| wordpress | Thiết lập các user và db cho wordpress, tải về wordpress và thiết lập cấu hình cho wordpress | defaults/main.yml | web_root<br>apache_name_service<br>http_name_service |

## Tree 

Cấu trúc của các role được thể hiện như sau: 

```
.
└── wplamp
    ├── apache_install
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   └── tasks
    │       └── main.yml
    ├── mysql_install
    │   ├── defaults
    │   │   └── main.yml
    │   ├── handlers
    │   │   └── main.yml
    │   ├── README.md
    │   └── tasks
    │       └── main.yml
    ├── php_config
    │   ├── defaults
    │   │   └── main.yml
    │   └── tasks
    │       ├── main.yml
    │       ├── php_install_centos7.yml
    │       └── php_install_ubuntu18.yml
    ├── README.md
    └── wordpress
        ├── defaults
        │   └── main.yml
        ├── handlers
        │   └── main.yml
        └── tasks
            ├── centos_create_db_user.yml
            ├── chmod_centos.yml
            ├── chmod_ubuntu.yml
            ├── configure.yml
            ├── main.yml
            └── ubuntu_create_db_user.yml
```

thư mục role có các nhiệm vụ tương ứng như: 

- `apache_install`: cài đặt và khởi động apache

- `mysql_install`: cài đặt và khởi động mariadb

- `php_config`: cài đặt php và các mở rộng của nó

- `wordpress`: tải về, giải nén, tạo database, tạo user và cấu hình wp-config

## Guide

Để bắt đầu sử dụng playbook này, bạn chỉ cần làm theo các bước sau: 

### 1. Clone repository

```
cd
git clone https://github.com/hungviet99/ansible_playbook_wordpress.git
```
### 2. Truy cập vào thư mục playbook

```
cd /root/ansible_playbook_wordpress
```
### 3. Cấu hình file inventory

Thêm vào các máy chủ được quản lý trong file `ansible_playbook_wordpress/hosts`. 

Ví dụ như sau: 

```
[wordpress]
web1 ansible_host=10.10.30.4 ansible_port=22 ansible_user=root ansible_ssh_pass=password
```

trong đó: 

- `ansible_host` là địa chỉ của máy chủ được quản lý

- `ansible_port` là port sử dụng để ssh vào máy chủ từ xa

- `ansible_user` là thông tin user sử dụng để đăng nhập ssh 

- `ansible_ssh_pass` là password của user đó

### 4. Cấu hình file playbook

Ban đầu ta sẽ có 1 file playbook như sau: 

```
---
- hosts: wordpress
  become: yes
  remote_user: root
  vars:
    wp_mysql_db: wordpress
    wp_mysql_user: wpuser
    wp_mysql_password: password
  roles:
    - {role: wplamp/php_config, tags: ['php_config']}
    - {role: wplamp/apache_install, tags: ['apache_install']}
    - {role: wplamp/mysql_install, tags: ['mysql_install']}
    - {role: wplamp/wordpress, tags: ['wordpress']}
```

Sử dụng `vi` hoặc `vim` để truy cập file trên với đường dẫn `/root/ansible_playbook_wordpress/playbook_wordpress.yml` và sửa 1 số thông tin sau:

- Sửa thông tin remote user:

Có thể sửa remote user `root` thành các user có quyền truy cập vào các host được quản lý.

- Sửa thông tin biến `vars`: 

    - Biến dành cho tên database: `wp_mysql_db`
    - Biến dành cho tên user: `wp_mysql_user`
    - Biến dành cho password của user: `wp_mysql_password`

### 5. Chạy playbook

Sau khi cấu hình các cài đặt cần thiết, bạn có thể chạy playbook như sau :

```
ansible-playbook -i hosts playbook_wordpress.yml
```

Ta sẽ nhận được đầu ra tương tự như sau: 

```

PLAY [wordpress] *************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************
ok: [host1-10.10.30.5]
ok: [host4-10.10.30.4]

TASK [wplamp/php_config : include_tasks] *************************************************************************************************************************************************
skipping: [host4-10.10.30.4]
included: /root/ansible_playbook_wordpress/roles/wplamp/php_config/tasks/php_install_centos7.yml for host1-10.10.30.5

...

TASK [wplamp/wordpress : include_tasks] **************************************************************************************************************************************************
skipping: [host1-10.10.30.5]
included: /root/ansible_playbook_wordpress/roles/wplamp/wordpress/tasks/chmod_ubuntu.yml for host4-10.10.30.4

TASK [wplamp/wordpress : chmod permission for apache] ************************************************************************************************************************************
changed: [host4-10.10.30.4] => (item=chown -R www-data:www-data /var/www/html/*)
changed: [host4-10.10.30.4] => (item=chmod -R 755 /var/www/html/*)
changed: [host4-10.10.30.4] => (item=mv /var/www/html/index.html /var/www/html/index.html.bk)

TASK [wplamp/wordpress : Start httpd service] ********************************************************************************************************************************************
skipping: [host4-10.10.30.4]
changed: [host1-10.10.30.5]

PLAY RECAP *******************************************************************************************************************************************************************************
host1-10.10.30.5          : ok=24   changed=18   unreachable=0    failed=0    skipped=6    rescued=0    ignored=0
host4-10.10.30.4          : ok=17   changed=10   unreachable=0    failed=0    skipped=8    rescued=0    ignored=0
```
Khi `failed` = 0 tức là không có lỗi xảy ra trong quá trình cài đặt. Quá trình chạy playbook hoàn tất !

### 6. Kết thúc cài đặt 

Mở trình duyệt web của bạn và điều hướng đến http://10.10.30.4 (hoặc IP máy chủ web của bạn) để hoàn tất cài đặt WordPress.

Author Information
------------------

This role was created in 2020 by Nguyen Viet Hung. 
