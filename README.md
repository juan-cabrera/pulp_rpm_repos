pulp_rpm_repos
==============

This role interacts with a pulp-server. It can create and manage rpm
repositories.

Install pulp-server using the
[pulp_ansible](https://github.com/pulp/pulp_ansible) project

This role has been tested this versions of pulp:

```yaml
    pulp_source_dir: "git+https://github.com/pulp/pulpcore.git@3.0.0rc2"
    pulp_plugin_source_dir: "git+https://github.com/pulp/pulpcore-plugin.git@0.1.0rc2"
    pulp_install_plugins:
      pulp-rpm:
        app_label: "rpm"
        source_dir: "git+https://github.com/pulp/pulp_rpm.git@3.0.0b3"
```

If you want to run the tasks of this role in a host different that the
pulp-server. User this variables when installing the server

```yaml
    pulp_api_host: 0.0.0.0
    pulp_content_bind: "0.0.0.0:24816"
```

This permit to access to the API interface from outside the pulp server

Requirements
------------

Install this packages in the host where the task are executed:

  - httpie
  - jq


Role Variables
--------------

To interact with the pulp server you need to set this variables

```yaml
    pulp_admin_user: admin
    pulp_default_admin_password: password
    pulp_api_server: http://localhost:24817
```

`pulp_admin_user`

To add and synchronize repositories you can add a list of different
repositories:

```yaml
    pulp_sync_repos:
      - repo: foo
        repo_version: "latest"
        remote: bar
        remote_url: https://repos.fedorapeople.org/pulp/pulp/fixtures/rpm-unsigned/
        policy: immediate
        base_path: arch/foo
        distribution: bar
        sync: yes
        state: present
      - repo: sync_epel
        repo_version: latest
        remote: epel
        remote_url: https://epel.mirror.it2go.eu/7/x86_64/
        policy: immediate
        base_path: common/sync_epel
        distribution: sync_epel
        sync: yes
        state: present
```

To add repositories with your own packages, add a list with repositories and
packages:

```yaml
    pulp_local_repos:
      - repo: repo_a_name
        repo_version: 2
        packages:
          - pkg1a-version.release.arch.rpm
          - pkg2a-version.release.arch.rpm
          - pkg3a-version.release.arch.rpm
        pkg_dir: /path1/to/your/packages/
        base_path: base path
        distribution: dist_a
        state: present
    
      - repo: repo_b_name
        repo_version: latest
        packages:
          - pkg1b-version.release.arch.rpm
          - pkg2b-version.release.arch.rpm
          - pkg3b-version.release.arch.rpm
        pkg_dir: /path2/to/your/packages/
        base_path: base/path
        distribution: distri_b
        state: present
```

Example Playbook
----------------

Create the following `pulp_api.ylm` playbook with the pulp server adress and password:

```yaml
    - hosts: localhost

      vars:
        pulp_admin_user: admin
        pulp_default_admin_password: password
        pulp_api_server: http://pulp-server:24817
        pulp_sync_repos:
        - repo: foo
          repo_version: "latest"
          remote: bar
          remote_url: https://repos.fedorapeople.org/pulp/pulp/fixtures/rpm-unsigned/
          policy: immediate
          base_path: arch/foo
          distribution: bar
          sync: yes
          state: present

      roles:
         - { role: pulp_rpm_repos }
```

And execute:

```sh
     ansible-playbook -c local playbooks/pulp_api.yml
```

License
-------

GPLv2

Author Information
------------------

Juan Cabrera at the [UNamur](https://www.unamur.be/)

