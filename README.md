Ansible Role: squid
=========

This role aims to configure Squid on Ubuntu 16.04 LTS, and hopefully later LTS releases.

In addition, and the main inspiration for creating this role, we integrate support for the (at the time of writing) universe package `squid-deb-proxy`.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

We set `visible_hostname` to whatever `ansible_hostname` is set to.

    squid_hostname: "{{ ansible_hostname }}"

In `squid_allowed_networks` we define a list of networks allowed to connect.
We default to IPv4 RFC1918, IPv6 link-local and localhost for both families.

    squid_allowed_networks:
      - 10.0.0.0/8
      - 172.16.0.0/12
      - 192.168.0.0/16
      - 127.0.0.1
      - fe80::/64
      - ::1

Set this to true if you want a proxy optimised for handling Ubuntu mirrors rather than a regular proxy.

    squid_deb_proxy: false

These are default values for the `squid_deb_proxy` parts and doesn't apply when the variable `squid_deb_proxy` is set to false.
`squid_mirror_custom` should be a list. Remember to remove `[]` if you want to add to it.

    squid_mirror_default: true
    squid_mirror_3rdparty: true
    squid_mirror_changelogs: true
    squid_mirror_ppas: false
    squid_mirror_custom: []


Except for these defined variables, there are some with default values in the templates that you can customise.
We list them here with their default values in common variable stanza:

What port should we listen on.

    squid_http_port: 3128

How large objects should we cache? Default is for squid-deb-proxy.

    squid_maximum_object_size: "512 MB"

Set up a `cache_dir` if you want. Normally Squid cache in memory, but the defaults listed are for squid-deb-proxy.
`squid_cache_size` is in MB, so 40GB.

    squid_cache_type: 'aufs'
    squid_cache_path: '/var/spool/squid' 
    squid_cache_size: 40000
    squid_cache_options: '16 256'

How much memory we want to use, and how large objects we will store there.

    squid_cache_mem: "200 MB"
    squid_max_object_mem_size: "10240 KB"


Example Playbook
----------------

    - hosts: proxies
      vars:
        squid_deb_proxy: true
        squid_mirror_custom:
          - .ftp.acc.umu.se
        squid_cache_size: 8000
      roles:
        - vcc-caeit.squid

License
-------

GPLv2

Author Information
------------------

This role was created in 2018 by Nafallo Bjälevik, whilst doing consultancy work for [Volvo Cars Corporation](http://www.volvocars.com/).
