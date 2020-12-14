Ansible Role: squid
===================

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

For making Avahi broadcast your deb proxy on the network, set this to true. With `squid-deb-proxy-client` installed on the clients
your proxy will then be automatically used if available on the same multicast domain.

    squid_deb_proxy_avahi: false

These are default values for the `squid_deb_proxy` parts and doesn't apply when the variable `squid_deb_proxy` is set to false.
`squid_mirror_custom` should be a list. Remember to remove `[]` if you want to add to it.

    squid_mirror_default: true
    squid_mirror_3rdparty: true
    squid_mirror_changelogs: true
    squid_mirror_ppas: false
    squid_mirror_custom: []

Except for these defined variables, there are some with default values in the templates that you can customise.
We list them here with their default values in common variable stanza:

There is support for cache peers by using a list of dicts. By default this variable is unset. We include a set of examples with all the options here.
To find out values for these options, you should visit the Squid documentation. `query_port` will be set to the default based on `query_type` if not set.

    squid_cache_peers:
      - host: parent.example.org
        type: parent
        port: 8080
        query_type: htcp
        query_port: 54213
        options:
          - default
          - htcp
      - host: sibling.example.org
        type: sibling
        port: 3128
        query_type: icp
        options: no-proxy

If you don't want to specify the `squid_cache_peers`, we include some shortcuts which apply reasonable defaults for the other values above.

    squid_parent_proxies: http://parent.example.org:8080
    squid_sibling_proxies:
      - sibling.example.org
      - sibling.example.net:3129

Since different environments are different, we included `squid_direct_connections` to handle when we connect directly to the origin servers.
We always check for cache hits on our cache peers before we decide how to fulfil the request, which is when this option gets interesting.
`always` will always use a direct connection and fail when then doesn't work. `never` will always use a parent proxy for the request and fail
if that does not work. `prefer` will prefer to connect to the origin server, but fallback to the parent proxies in case of failure. `fallback`
will always try the parent proxies first, but use direct connections as fallback would that fail.
By default this is not set, which will use whatever default your Squid installation uses.

    squid_direct_connections: always never prefer fallback

What port should we listen on. If we are a cache sibling, we will use the defined port instead of the `squid_http_port` variable.

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

In case you need Squid to use different nameservers than the system you can define `squid_nameservers` as either a string
for a single one or a list for more than one. By default this option is unset, and we use the system nameserver(s).

    squid_nameservers: 127.0.0.53


Example Playbook
----------------

    - hosts: proxies
      vars:
        squid_deb_proxy: true
        squid_mirror_custom:
          - .ftp.acc.umu.se
        squid_cache_size: 8000
      roles:
        - vcc_caeit.squid

License
-------

GPLv2

Author Information
------------------

This role was created in 2018 by Nafallo Bjälevik, whilst doing consultancy work for [Volvo Cars Corporation](http://www.volvocars.com/).
