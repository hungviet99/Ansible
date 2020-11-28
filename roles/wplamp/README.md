# Wordpress Playbook

Cài đặt cấu hình máy chủ web dựa trên wordpress với Mariadb, PHP 7.2, apache2.

## Roles

| Role | Description | var path | var name | 
|-------|------------| -------- | -------- |
| php_config | Cài đặt PHP và các phần mở rộng PHP cần thiết mà Wordpress cần | defaults/main.yml | php_packages | 
| apache_install | Cài đặt apache2 làm web server | defaults/main.yml | apache_packages<br>apache_name_service |
| mysql_install | Cài đặt `mariadb-server`, `mariadb-client` và `python-mysqldb` | defaults/main.yml | mysql_python_package_debian<br>sql_name_service<br>mariadb_packages |
| wordpress | Thiết lập các user và db cho wordpress, tải về wordpress và thiết lập cấu hình cho wordpress | defaults/main.yml | web_root<br> apache_name_service |

## Requirements

- Clone repo 

```
git clone 



Example Playbook
----------------
    - hosts: servers
      become: yes
      vars:
        wp_mysql_db: wordpress
        wp_mysql_user: wpuser
        wp_mysql_password: password
      roles:
        - {role: wplamp/php_config, tags: ['php_config']}
        - {role: wplamp/apache_install, tags: ['apache_install']}
        - {role: wplamp/mysql_install, tags: ['mysql_install']}
        - {role: wplamp/wordpress, tags: ['wordpress']}

Author Information
------------------

This role was created in 2020 by Nguyen Viet Hung. 
