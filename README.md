# Ansible collection - `vladgh.samba`

![Test Status](https://github.com/vladgh/ansible-collection-vladgh-samba/actions/workflows/test.yml/badge.svg)

An Ansible collection for setting up Samba as a file server or a domain controller (primary or secondaries). It is tested on Ubuntu, Debian, CentOS and Arch Linux. Specifically, the responsibilities of this collection are to:

- Install the necessary packages
- Create share directories
- Manage Samba users and passwords
- Manage access to shares

The following are not considered concerns of this collection, and you should configure these using another collection:

- Managing firewall settings.
- Creating system users. Samba users should already exist as system users.

**If you like/use this collection, please consider giving it a star! Thanks!**

## CVE-2017-7494

A remote code execution vulnerability may affect your Samba server installation. Samba versions 3.5.0 and before 4.6.4 are affected. If SELinux is enabled on your system, it is **NOT** vulnerable.

This collection will check if the installed version of Samba is affected by the vulnerability and apply the proposed workaround: adding `nt pipe support = no` to the `[global]` section of the configuration. Remark that this disables share browsing by Windows clients.

You can explicitly disable the fix if necessary, by setting the server role variable `samba_mitigate_cve_2017_7494` to `false`.

More info: <https://access.redhat.com/security/cve/cve-2017-7494>

## Using this collection

### Install collection

Install it with the Ansible Galaxy CLI:

```sh
ansible-galaxy collection install vladgh.samba  --upgrade
```

You can also include it in a `requirements.yml` file and install it via `ansible-galaxy collection install -r requirements.yml`, using the format:

```yaml
---
collections:
  - name: vladgh.samba
```

Optionally, You can specify a version number:

```yaml
---
collections:
  - name: vladgh.samba
    version: ">=3.1.1"
```

You can also refer to a branch or to a git commit-ish object (commit, tag, branch or release)

```yaml
collections:
  - name: https://github.com/vladgh/ansible-collection-vladgh-samba
    version: main
    type: git
```

For more information, see [Ansible using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)

### Load role

After the collection is installed you can include the `server` role.
You can define variables in a variety of places, such as in inventory, in playbooks, in reusable files, in roles, and at the command line. Ansible loads every possible variable it finds, then chooses the variable to apply based on [variable precedence rules](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#ansible-variable-precedence).
A complete overview of server role variables follows below.

For more information, see [Using variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#where-to-set-variables).

```yaml
---
- name: Samba Server
  hosts: all
  become: true
  tasks:
    - name: Include Samba Server role
      ansible.builtin.include_role:
        name: vladgh.samba.server
```

### Import playbook

Alternatively, you can directly import the existing playbook:

```yaml
---
- name: Samba Server
  ansible.builtin.import_playbook: vladgh.samba.server
```

## Server Role Variables

| Variable                       | Default                  | Comments                                                                                                                     |
| :---                           | :---                     | :---                                                                                                                         |
| `samba_apple_extensions`       | `true`                       | When true, enables support for Apple specific SMB extensions. Required for Time Machine support to work (see below)       |
| `samba_create_varwww_symlinks` | `false`                    | When true, symlinks are created in web docroot to the shares. (`var/www/` or `/var/www/html` depending on platform) |
| `samba_cups_server`            | `localhost:631`            | Value for the global option `cups server` (only needed when `samba_printer_type` is "cups")                                  |
| `samba_enable_netbios`         | `true`                   | When false, the NMB daemon is disabled by setting `disable netbios` to `yes`. This overrides other NetBIOS related settings. |
| `samba_domain_master`          | `true`                     | When true, smbd enables WAN-wide browse list collation                                                                       |
| `samba_global_include`         | -                        | Samba-compatible configuration file with options to be loaded to [global] section (see below)                                |
| `samba_guest_account`          | -                        | Guest account for unknown users                                                                                              |
| `samba_homes_include`          | -                        | Samba-compatible configuration file with options to be loaded to [homes] section (see below)                                 |
| `samba_interfaces`             | `[]`                       | List of network interfaces used for browsing, name registration, etc.                                                        |
| `samba_load_homes`             | `false`                    | When true, user home directories are accessible.                                                                             |
| `samba_load_printers`          | `false`                    | When true, printers attached to the host are shared                                                                          |
| `samba_local_master`           | `true`                     | When true, nmbd will try & become local master of the subnet                                                                 |
| `samba_log`                    | -                        | Set the log file. If left undefined, logging is done through syslog.                                                         |
| `samba_log_size`               | `5000`                     | Set the maximum size of the log file.                                                                                        |
| `samba_log_level`              | `0`                        | Set Samba log level, 0 is least verbose and 10 is a flood of debug output.                                                   |
| `samba_map_to_guest`           | `Never`                  | Behaviour when unregistered users access the shares.                                                                         |
| `samba_mitigate_cve_2017_7494` | `true`                     | CVE-2017-7494 mitigation breaks some clients, such as macOS High Sierra.                                                     |
| `samba_mdns_name`              | `netbios`                  | The name advertised via multicast DNS.                                                                                             |
| `samba_netbios_name`           | `{{ ansible_hostname }}` | The NetBIOS name of this server.                                                                                             |
| `samba_passdb_backend`         | `tdbsam`                 | Password database backend.                                                                                                   |
| `samba_preferred_master`       | `true`                     | When true, indicates nmbd is a preferred master browser for workgroup                                                        |
| `samba_realm`                  | -                        | Realm domain name                                                                                                            |
| `samba_printer_type`           | `cups`                     | value for the global option `printing` and `printcap name`                                                                   |
| `samba_security`               | `user`                   | Samba security setting                                                                                                       |
| `samba_server_max_protocol`    | -                        | Specify a maximum protocol version offered by the server.                                                                    |
| `samba_server_min_protocol`    | -                        | Specify a minimum protocol version offered by the server.                                                                    |
| `samba_server_string`          | `fileserver %m`          | Comment string for the server.                                                                                               |
| `samba_shares_root`            | `/srv/shares`            | Directories for the shares are created under this directory.                                                                 |
| `samba_manage_directories`     | `true`                   | Create the directories, and manage the permissions/ownership, of the shares root and the shares under it.                    |
| `samba_shares`                 | `[]`                       | List of dicts containing share definitions. See below for details.                                                           |
| `samba_username_map`           | `[]`                       | Makes username map configurable.                                                                         |
| `samba_users`                  | `[]`                       | List of dicts defining users that can access shares.                                                                         |
| `samba_wins_support`           | `true`                     | When true, Samba will act as a WINS server                                                                                   |
| `samba_workgroup`              | `WORKGROUP`              | Name of the server workgroup.                                                                                                |

### Defining users

In order to allow users to access the shares, they need to get a password specifically for Samba:

```yaml
samba_users:
  - name: alice
    password: ecila
  - name: bob
    password: bob
  - name: charlie
    password: eilrahc
```

It is recommended to [encrypt the password content with Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html). Also, remark that this collection will not change the password of an existing user.

These users should already have an account on the host! Creating system users is not a concern of this collection, so you should do this separately:

```yaml
- name: Add the user 'james' with a bash shell, appending the group 'admins' and 'developers' to the user's groups
  ansible.builtin.user:
    name: james
    shell: /bin/bash
    groups: admins,developers
    append: true
```

### Defining shares

Defining Samba shares and configuring access control can be challenging, since it involves not only getting the Samba configuration right, but also user and file permissions, and SELinux settings. This collection attempts to simplify the process.

To specify a share, you should at least give it a name:

```yaml
samba_shares:
  - name: readonlyshare
```

This will create a share with only read access for registered users. Guests will not be able to see the contents of the share.

A good way to configure write access for a share is to create a system user group, add users to that group, and make sure they have write access to the directory of the share. This collection assumes groups are already set up and users are members of the groups that control write access. Let's assume you have two users `jack` and `teach`, members of the group `pirates`. This share definition will give both read and write access to the `pirates`:

```yaml
samba_shares:
  - name: piratecove
    comment: 'A place for pirates to hang out'
    group: pirates
    write_list: +pirates
```

Guests have no access to this share, registered users can read. You can further tweak access control. Read access can be extended to guests (add `public: true`) or restricted to specified users or groups (add `valid_users: +pirates`). Write access can be restricted to individual pirates (e.g. `write_list: jack`). Files added to the share will be added to the specified group and group write access will be granted by default.

This is an example of configuring multiple vfs object modules to share a glusterfs volume. VFS object options are optional. The necessary VFS object modules must be present/installed outside this collection. In this case samba-glusterfs was installed on centos. See samba documentation for how to install or what the default VFS object modules are.

```yaml
samba_shares:
  - name: gluster-app_deploys
    comment: 'For samba share of volume app_deploys'
    vfs_objects:
      - name: audit
        options:
          - name: facility
            value: LOCAL1
          - name: priority
            value: NOTICE
      - name: glusterfs
        options:
          - name: volume
            value: app_deploys
          - name: logfile
            value: /var/log/samba/glusterfs-app_deploys.%M.log
          - name: loglevel
            value: 7
    path: /
    read_only: false
    guest_ok: true
    write_list: tomcat
    group: tomcat
```

A complete overview of share options follows below. Only `name` is required, the rest is optional.

| Option                 | Default                         | Comment                                                                                        |
| :---                   | :---                            | :---                                                                                           |
| `browsable`           | -                               | Synonim `browseable`.                                           |
| `browseable`           | -                               | Controls whether this share appears in file browser.                                           |
| `comment`              | -                               | A comment string for the share                                                                 |
| `create_mode`          | `0664`                          | See the Samba documentation for details.                                                       |
| `directory_mode`       | `0775`                          | See the Samba documentation for details.                                                       |
| `include_file`         | -                               | Samba combatible configuration file with options to be included for this share (see below).    |
| `force_create_mode`    | `0664`                          | See the Samba documentation for details.                                                       |
| `force_directory_mode` | `0775`                          | See the Samba documentation for details.                                                       |
| `group`                | `users`                         | The user group files in the share will be added to. (force group)                              |
| `guest_ok`             | -                               | Allow guest access.                                                                            |
| `name` (required)      | -                               | The name of the share.                                                                         |
| `owner`                | `root`                          | Set the owner of the path                                                                      |
| `path`                 | `/{{samba_shares_root}}/{{name}}` | The path to the share directory.                                                               |
| `public`               | `false`                            | Controls read access for guest users                                                           |
| `read_only`               | -                            | If this parameter is yes, then users of a service may not create or modify files in the service's directory.                        |
| `setype`               | -                 | The SELinux type of the share directory                                                        |
| `user`                 | -                               | The user files in the share will be added to. (force user)                                     |
| `valid_users`          | -                               | Controls read access for registered users. Use the syntax of the corresponding Samba setting.  |
| `vfs_objects`          | -                               | See the Samba documentation for details.                                                       |
| `writable`             | -                               | Synonym for `writeable`.                                                                           |
| `writeable`             | -                               | Writeable for guests.                                                                           |
| `write_list`           | -                               | Controls write access for registered users. Use the syntax of the corresponding Samba setting. |

The values for `valid_users` and `write_list` should be a comma separated list of users. Names prepended with `+` or `@` are interpreted as groups. The documentation for the [Samba configuration](https://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html) has more details on these options.

## Username mapping

Set `samba_username_map` variable as follows:

```yaml
samba_username_map:
  - { from: 'UserOnWindows', to: 'userOnLinux' }
  - { from: 'foo', to: 'bar' }
```

which is equal to username map as follows:

```ini
userOnLinux = UserOnWindows
bar = foo
```

## Adding arbitrary configuration files

You can add settings that are not supported by this collection out-of-the-box through custom configuration files that will be included from the main configuration file. There are three types of include files: for the global section, for the homes section, and for individual shares. Put your custom configuration files in a subdirectory `templates`, relative to your master playbook location. Then, specify them in the variables `samba_global_include`, `samba_homes_include`, or `include_file` in the `samba_shares` definition.

Your custom configuration files are considered to be Jinja templates, so you can use Ansible variables inside them. The configuration files will be validated to ensure they are syntactically correct.

For example, to include `templates/global-include.conf`, set:

```yaml
samba_global_include: global-include.conf
```

Remark that is it not necessary to specify the `templates/` directory.

Likewise, to include `templates/piratecove-include.conf`, specific for the `piratecove` share (see the example above); set:

```yaml
samba_shares:
  - name: piratecove
    comment: 'A place for pirates to hang out'
    group: pirates
    write_list: +pirates
    include_file: piratecove-include.conf
```

The [test playbook](molecule/default/converge.yml) has some examples.

## Dependencies

* Fileserver: N/A
* Domain Controller: DNS must be setup separately and match the target realm, domain/subdomain. NTP must be setup separately.

## Testing

This collection is tested using [Ansible Molecule](https://molecule.readthedocs.io/). Tests are launched automatically via [GitHub Actions](https://github.com/vladgh/ansible-collection-vladgh-samba/actions) after each commit and PR.

This Molecule configuration will:

- Create a Docker container
- Run a syntax check
- Apply the role with a [test playbook](molecule/default/converge.yml)
- Run idempotence check

This process is repeated for the supported Linux distributions.

### Local test environment

Steps to install the necessary tools manually:

1. Docker and Python >= 3.8 should be installed on your machine (assumed to run Linux).
2. As recommended by Molecule, create a python virtual environment
3. Install the software tools `python3 -m pip install --user --upgrade pip ansible ansible-lint molecule docker`
4. Navigate to the root of the collection directory and run `molecule test`

Molecule automatically deletes the containers after a test. If you would like to check out the containers yourself, run `molecule converge` followed by `molecule login --host HOSTNAME`.

The Docker containers are based on images created by [Jeff Geerling](https://hub.docker.com/u/geerlingguy), specifically for Ansible testing (look for images named `geerlingguy/docker-DISTRO-ansible`). You can use any of his images, but only the distributions mentioned in [meta/main.yml](meta/main.yml) are supported.

The default config will start an Ubuntu 22.04 container. Choose another distro by setting the `MOLECULE_DISTRO` variable with the command, e.g.:

```bash
MOLECULE_DISTRO=debian11 molecule test
```

## Troubleshooting and Known issues

* "INTERNAL ERROR: Security context active token stack underflow! in  () () pid 41380 (4.19.5-Ubuntu)":
Samba Domain Controller provisioning has issues running inside unprivileged lxc container. Workaround is to use privileged container (https://forum.proxmox.com/threads/error-samba-ad-provisioning-into-lxc-container.33475/, https://github.com/turnkeylinux/tracker/issues/1666). Alternate solutions that may apply or not, either use nfs attributes to store acl (https://lists.samba.org/archive/samba/2021-February/234326.html), either tweak samba uid along alternate filesystem like btrfs (https://github.com/lxc/lxc/issues/2708#issuecomment-473466062).

## Changelog & Releases

This repository keeps a change log using [GitHub's releases](https://github.com/vladgh/ansible-collection-vladgh-samba/releases) functionality.

Releases are based on [Semantic Versioning](https://semver.org/), and use the format
of `MAJOR.MINOR.PATCH`. The version will be incremented
based on the following:

- `MAJOR`: Incompatible or major changes
- `MINOR`: Backwards-compatible new features and enhancements
- `PATCH`: Backwards-compatible bugfixes and package updates

## Contribute

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](.github/CODE_OF_CONDUCT.md)

Contributions are always welcome! Please read the [contribution guidelines](.github/CONTRIBUTING.md) and the [code of conduct](.github/CODE_OF_CONDUCT.md).

## Contributors

This collection could only have been realized thanks to the contributions of the people listed below. If you have an idea to improve it even further, don't hesitate to pitch in! Please create a topic branch for your proposed changes. If you don't, this will create conflicts in your fork after the merge. Don't hesitate to add yourself to the contributor list below in your PR!

[Ben Tomasik](https://github.com/tomislacker),
[Bengt Giger](https://github.com/BenGig),
[Bert Van Vreckem](https://github.com/bertvv/) (maintainer),
[Birgit Croux](https://github.com/birgitcroux),
[DarkStar1973](https://github.com/DarkStar1973),
[George Hartzell](https://github.com/hartzell),
[Ian Young](https://github.com/iangreenleaf),
[Jonas Heinrich](https://github.com/onny),
[Jonathan Underwood](https://github.com/jonathanunderwood),
[Karl Goetz](https://github.com/goetzk),
[morbidick](https://github.com/morbidick),
[Paul Montero](https://github.com/lpaulmp),
[Robin Ophalvens](https://github.com/RobinOphalvens),
[Slavek Jurkowski](https://github.com/slavekjurkowski2),
[Sven Eeckeman](https://github.com/SvenEeckeman),
[Tiemo Kieft](https://github.com/blubber),
[Tobias Wolter](https://github.com/towo),
[Tomohiko Ozawa](https://github.com/kota65535),
[Vlad Ghinea](https://github.com/vladgh/) (maintainer),

## License

2-clause BSD license, see [LICENSE.md](LICENSE.md)
