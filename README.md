Ansible Role: squid
===================

This role aims to configure Squid on Ubuntu 16.04 LTS, and hopefully later LTS releases.

In addition, and the main inspiration for creating this role, we integrate support for the (at the time of writing) universe package `squid-deb-proxy`.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

We set `visible_hostname` to whatever `ansible_host` is set to.

    squid_hostname: "{{ ansible_host }}"

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
`squid_mirror_debian` and `squid_mirror_ubuntu` defaults to true or false depending on the host you're setting up Squid on, with
the other options except for `squid_mirror_custom` depending on these settings. `squid_mirror_custom` should be a list.

    squid_mirror_debian: true on Debian servers, false otherwise
    squid_mirror_ubuntu: true on Ubuntu servers, false otherwise
    squid_mirror_default: true
    squid_mirror_3rdparty: true
    squid_mirror_changelogs: true
    squid_mirror_ppas: false
    squid_mirror_custom: []

If you are not sure which repositories your users will use, you can set `squid_mirror_match_all_repositories` to true.
This will try and regex match the path of the URL to valid repository formats.
Since regex is slow we add this as a last hit and default to not include this.

    squid_mirror_match_all_repositories: false

When OpenDNS/Cisco Umbrella automatic redirection is in use on your network, you will likely need the cache server to support the redirects.

    squid_mirror_opendns: false

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

In some cases you need to do some external adaptions to what traffic gets served. Squid supports ICAP (Internet Content Adaptation Protocol) for this.
You can define services per the example, although only `name` is required if you are happy with the defaults. `bypass` will not get set, unless defined, and will use the Squid default value.
This role will not take care of setting up the ICAP service(s).

    squid_icap_policy:
      - name: service
        direction: in
        bypass: false
        url: http://127.0.0.1:1344/service
        access:
          - acl: all
            allow: true

What port should we listen on. If we are a cache sibling, we will use the defined port instead of the `squid_http_port` variable for `squid_deb_proxy`.
When `squid_deb_proxy` is not set, there is now support for having this variable be a list of Listen directives.

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
