# Nginx HTTP & HTTPS static web server

[![Build Status](https://travis-ci.org/Universum/ansible-role-nginx-www.svg?branch=master)](https://travis-ci.org/Universum/ansible-role-nginx-www)

## What does it do?

1. Installs Nginx
2. Creates an SSL certificate and DH parameter store location as defined in {{nginx_proxy_ssl_store_path}}
3. Generates secure DHparams (2048)
4. Copies all the SSL certificates and keys defined in ssl sites to your SSL store
5. Copies the custom Nginx configuration with only secure ciphers
6. Copies all of your virtual site configurations defined in defaults/main or {group,host}_vars to sites-available
7. Symlinks all of the above virtual sites to sites-enabled

Your sites content should be in /www/{{item.site_id}}!

## What do you have to do?

1. Take a look at the defaults/main file to look at the available option for virtual sites
2. Move the defaults/main file to your group_vars or host_vars (you can of course edit in place if you wish, but it isn't recommended)
3. Look at the example playbooks and read the rest of the documentation

It is recommended that you put your certificates and keys into a separate vault-encrypted file and only reference them in the regular vars or defaults file, in order to protect them

Remember, you probably want to remove or comment out the defaults/main.yml file before running 


nginx_www_ssl_virtual_sites:

Name | Example | Required | Use
--- | --- | --- | ---
site_id | example | Yes | will be the name for upstream server and the sites-available file, must be unique
http_port | 80, | No (default: 80) *ignored if custom_http_block is used* | http port for nginx to listen to
https_port | 443 | No (default: 443) *ignored if custom_https_block is used* | https port for nginx to listen to
server_names | example.com example2.com | Yes | server names to listen for
ssl_cert | ---BEGIN CERTIFICATE---... | Yes, if https_port is defined and ssl_*_path is not defined | SSL certificate
ssl_key: | ---BEGIN PRIVATE KEY---... | Yes, if https_port is defined and ssl_*_path is not defined | SSL key
ssl_cert_path | /var/ssl/cert.cert | Yes, if https_port is defined and ssl_* is not defined| SSL certificate path on remote sys
ssl_key_path | /var/ssl/key.key | Yes, if https_port is defined and ssl_* is not defined| SSL key on remote sys
letsencrypt | true/false | No | sets up letsencrypt store for your server (site_id must match servername)
ssl_protocols | 'TLSv1 TLSv1.1 TLSv1.2' | No (default: 'TLSv1 TLSv1.1 TLSv1.2') | ssl protocols we should accept for this site 
index | index.html index.htm index.php | Yes | index to serve
custom_http_block: | server { listen 80; etcetc } | No | here we can substitute the standard http->https redirect block with a custom server {} block if we want, will negate all other HTTP options
custom_https_block: | server { listen 443; etcetc } | No | here we can substitute the standard http->https redirect block with a custom server {} block if we want, will negate all other HTTP options
```

``` ansible-playbook example-playbook.yml ```

so that it doesn't try to set up my example sites.

```
#example-playbook.yml
- name: example.com
  hosts: all
  vars:
    nginx_www_virtual_sites: 
      - { site_id: 'example.com', index: 'index.html' }
  roles:
    - nginx-www
  tasks:
    - file: 
        path: /www/example.com 
        state: directory
        owner: "{{nginx_user}}"
        group: "{{nginx_group}}"
        mode: 0755
        setype: httpd_config_t
      become: yes
    - copy:
        src: ../files/index.html
        dest: /www/example.com/index.html
        owner: "{{nginx_user}}"
        group: "{{nginx_group}}"
        mode: 0755
        setype: httpd_config_t
      become: yes
```

Compability
-----

Tested on:

Debian 8 & CentOS 7
