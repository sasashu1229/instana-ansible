# Install Db2

Ansible role that installs Db2.

## Requirements

1. Download Db2

   Click on the [Db2 11.5 Mod 7 FixPack 0 link from Download DB2 Fix Packs by version for DB2 for Linux, UNIX and Windows](https://www.ibm.com/support/pages/download-db2-fix-packs-version-db2-linux-unix-and-windows) to download the Db2 Server Fix Pack for Linux 64-bit, x86-64 on AMD64 and Intel EM64T.

2. Place the downloaded file (e.g `v11.5.7_linuxx64_server_dec.tar.gz`) in `./roles/setup_db2/files` directory

## Note

- specify `--ask-vault-pass` when start this role, then enter a password.
- set variable name '<username>\_password' for os user password.
  - db2 uses username 'db2inst1' and 'db2fenc1' as a default .

<!-- Role Name

A brief description of the role goes here.

## Requirements

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 } -->

## License

BSD

## Author Information

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
