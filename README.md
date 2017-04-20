[![Build Status](https://travis-ci.org/CSCfi/ansible-role-jetty.svg?branch=master)](https://travis-ci.org/CSCfi/ansible-role-jetty)

Ansible-Role: Jetty
=========

An role which downloads and unpacks Jetty under /opt. Jetty user and group are also created if needed.

Requirements
------------

None. Purpose of this role is to perform base installation.

Role Variables
--------------

See defaults/main.yml for the variables you can overwrite via role call as a parameter.

* jetty_version: 9.4.2.v20170220
* jetty_dst_dir: /opt
* jetty_user_homedir: /opt/jetty

* jetty_user_name: jetty
* jetty_group_name: jetty

* jetty_group_create: true
* jetty_user_create: true

Dependencies
------------

None

Example Playbook
----------------

    - hosts: all
      roles:
        - { role: CSCfi.jetty }



