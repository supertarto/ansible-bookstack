# Ansible Bookstack
[![Build Status](https://travis-ci.com/supertarto/ansible-bookstack.svg?branch=master)](https://travis-ci.com/supertarto/ansible-bookstack)

Install and configure Bookstack with Ansible.

## Tested plateform
* Debian 10 (Buster)

## Role variables
Set this variable to true to force the application update or reinstallation.
```yml
bookstack_force_update: false
```
Set this variable to true if you need to update your .env file. Default value set to false for idempotence.
```yml
bookstack_force_env_update: false
```
Informations about your local git directory, on your server. We use a separate  git directory. You can alse define the remote git repository and branch to clone.
```yml
bookstack_separate_git_dir: /usr/local/bookstack-git
bookstack_content_dest: /var/www/Bookstack
bookstack_git_branch_version: release
bookstack_git_url: https://github.com/BookStackApp/BookStack.git
```
Your databases informations, used in your .env file.
```yml
bookstack_host: localhost
bookstack_db_name: bookstackdb
bookstack_db_user: bookstackuser
bookstack_db_password: bookstackpassword
```
SMTP and mail informations.
```yml
bookstack_mail_driver: smtp
bookstack_mail_from_name: BookStack
bookstack_mail_from: bookstack@myentreprise.com

bookstack_smtp_host: localhost
bookstack_smtp_port: "1025"
bookstack_smtp_username: "null"
bookstack_smtp_password: "null"
bookstack_smtp_encryption: "null"
```

## Examples
```yml
- hosts: all
  roles:
    - role: supertarto.apache
    - role: supertarto.php
    - role: supertarto.mariadb
    - role: supertarto.bookstack

  pre_tasks:
    - name: Update apt cache.
      apt:
        update_cache: true
        cache_valid_time: 600
      when: ansible_os_family == 'Debian'
      changed_when: false
  vars:
    php_packages:
      - php7.3
      - php7.3-mysql
      - php7.3-curl
      - php7.3-pdo
      - php7.3-xml
      - php7.3-mbstring
      - php7.3-gd
      - php-tokenizer
      - php7.3-tidy

    bookstack_host: localhost
    bookstack_db_name: bookstackdb
    bookstack_db_user: bookstackuser
    bookstack_db_password: bookstackpassword

    apache_create_vhosts: true
    apache_mods_enabled:
      - rewrite

    apache_vhosts_filename: "bookstack.conf"
    apache_vhost_config:
      - listen_ip: "*"
        listen_port: 80
        server_name: host1
        documentroot: "/var/www/Bookstack/public"
        serveradmin: admin@localhost
        custom_param: |
          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
          LogLevel warn
        directory:
          - path: "/var/www/Bookstack/public"
            config: |
              AllowOverride All
              Order deny,allow
              allow from all
              Options +FollowSymLinks
              RewriteEngine On
              RewriteCond %{REQUEST_FILENAME} !-d
              RewriteCond %{REQUEST_FILENAME} !-f
              RewriteRule ^ index.php [L]

    mariadb_use_dump_script: false
    mariadb_databases:
      - name: "{{ bookstack_db_name }}"

    mariadb_users:
      - name: "{{ bookstack_db_user }}"
        host: "{{ bookstack_host }}"
        password: "{{ bookstack_db_password }}"
        priv: "{{ bookstack_db_name }}.*:SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,CREATE TEMPORARY TABLES,LOCK TABLES"
```

## Installation
```
ansible-galaxy install supertarto.bookstack
```
## License
GPL V3.0
