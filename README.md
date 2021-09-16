packages
========

This role handles installed packages, repositories and pinning.

Version
-------

* `2.3.0` --- add rhel8 support + remove trusty
* `2.2.2` --- adding missing dependency on `update-notifier-common`
* `2.2.1` --- fix linting, remove testing for ubuntu precise
* `2.2.0` --- added ubuntu focal, 20.04
* `2.1.2` --- tested with Ansible 2.9.11
* `2.1.1` --- prepare for github
* `2.1.0` --- adding `packages_services_enabled` and `packages_services_disabled` to start+enable or stop+disable services
* `2.0.3` --- fixed spelling error on `packages_centos_repositories` variable
* `2.0.2` --- fixed check mode when unattended upgrades is not installed
* `2.0.1` --- fixing variable names
* `2.0.0` --- auto update security packages, optional reboot on kernel change
* `1.0.1` --- fix various typos
* `1.0.0` --- initial version
* `master` --- latest development version

Requirements
------------

This role is limited to

* Ubuntu 20.04 - Bionic
* Ubuntu 18.04 - Bionic
* Ubuntu 16.04 - Xenial
* CentOS 8
* CentOS 7
* RHEL 8

Role Variables
--------------


* `packages_install` --- list of packages to install on Ubuntu or CentOS, default `[]`.
* `packages_unattended_upgrades` --- automatic upgrade security packages, default `true`
* `packages_only_security_upgrades` --- only update security patches else all packages, default `true`
* `packages_automatic_reboot` --- enable automatic reboot if needed after update, default `false`
* `packages_automatic_reboot_hour` --- hour to reboot if needed, default `3`
* `packages_automatic_reboot_minute` --- minute of hour to reboot if needed, default `13`
* `packages_services_enabled` --- list of services to start and enable at boot time, default `[]`
* `packages_services_disabled` --- list of services to stop and disable at boot time, default `[]`

### Ubuntu specific

* `packages_ubuntu_install` --- list of packages to install on Ubuntu, default `[]`.
* `packages_ubuntu_repositories` --- list of dictionaries with repositories to add, `{}`. See below for dict elements.
    * `repo` --- full repository line to repository, __mandatory__.
    * `enabled` --- boolean value with should this repository be enabled or disabled, __mandatory__.
    * `key_url` --- URL to ASCII armored key for repository, default undefined.
    * `key_id` --- PGP key ID - only needed for Ubuntu repositories, default undefined.
* `packages_ubuntu_pin` --- list dicts of packges to pin on Debian, default `{}`.
    * `package` --- packages to pin, default not set.
    * `pin` --- pin to, default not set.
    * `pin_priority` --- priority, default not set.
* `packages_ubuntu_hold` --- list of packages to hold in Ubuntu or exclude on CentOS - CentOS accepts globbing, default `[]`.
* `packages_ubuntu_unhold` --- list of packages to remove hold on in Ubuntu, default `[]`.
* `packages_ubuntu_update_backports` --- update backports during unattended upgrades, default `false`

### CentOS/RHEL specific

* `packages_centos_install` --- list of packages to install on CentOS or RHEL, default `[]`.
* `packages_centos_repositories` --- list of dictionaries with repositories to add, `{}`.
    * `name` --- name of repository, __mandatory__.
    * `description` --- repository description, __mandatory__.
    * `baseurl` --- URL to Ubuntu or CentOS repository, __mandatory__.
    * `enabled` --- boolean value with should this repository be enabled or disabled, __mandatory__.
    * `gpgkey` --- URL to ASCII armored key for repository, default `''`.
* `packages_centos_exclude` --- list of package gobs to exclude, default `[]`.

Dependencies
------------

The RHEL8 image needs to be registered with RedHat to install packages.

Example Playbook
----------------

    - hosts: servers
      roles:
         - role: packages
           packages_install:
             - vim
             - nmap
           packages_ubuntu_install:
             - emacs
           packages_ubuntu_repositories:
             - repo: deb https://apt.kubernetes.io/ kubernetes-xenial main
               key_url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
               key_id: 54A647F9048D5688D7DA2ABE6A030B21BA07F4FB
               enabled: true
             - repo: deb https://linux.dell.com/repo/community/openmanage/940/bionic bionic main
               key_url: https://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc
               key_id: 1285491434D8786F
               enabled: true
           packages_ubuntu_pin:
             - package: kubectl
               pin: 'release n={{ ansible_distribution_release }}'
               priority: 10
           packages_ubuntu_hold:
             - emacs
             - vim
           packages_ubuntu_unhold:
             - nano
           packages_centos_install:
             - emacs
           packages_centos_repositories:
             - name: kubernetes
               description: kubernetes repository
               baseurl: https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
               gpgkey: https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
               enabled: true
             - name: dell-system-update_os_independent
               description: dell system update
               baseurl: https://linux.dell.com/repo/hardware/dsu/os_independent/
               gpgkey: https://linux.dell.com/repo/pgp_pubkeys/0x756ba70b1019ced6.asc https://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc https://linux.dell.com/repo/pgp_pubkeys/0xca77951d23b66a9d.asc
               enabled: true
           packages_centos_exclude:
             - nano
           packages_only_security_upgrades: false
           packages_ubuntu_update_backports: true
           packages_automatic_reboot: true

Testing
-------

To test RHEL8 with vagrant, install `vagrant-register`

```bash
vagrant plugin install vagrant-registration
```

### Test environment for all OSes

```bash
cd tests
vagrant up
```

### Rerun role

Run role on all OSes again.

```bash
vagrant provision
```

### Debug interactively

This uses cluster ssh to work with all vagrant boxes at the same time.

```bash
vagrant ssh-config > ~/.ssh/config
cat ~/.ssh/config | grep ^Host | cut -d\  -f2 | xargs cssh
```

License
-------

GPLv2

Author Information
------------------

Created 2020 by [Arnulf Heimsbakk](mailto:arnulf.heimsbakk@met.no) for MET Norway.

### References

* https://help.ubuntu.com/community/PinningHowto

###### set vim: spell spelllang=en:
