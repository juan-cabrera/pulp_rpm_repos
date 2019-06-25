pulp_rpm_repos
==============

This role interacts with a [pulp-server](https://pulpproject.org/). It helps to
create and manage rpm repositories.

You can install the pulp-server using the
[pulp_ansible](https://github.com/pulp/pulp_ansible) project

This role has been tested with this versions of pulp:

```yaml
    pulp_source_dir: "git+https://github.com/pulp/pulpcore.git@3.0.0rc2"
    pulp_plugin_source_dir: "git+https://github.com/pulp/pulpcore-plugin.git@0.1.0rc2"
    pulp_install_plugins:
      pulp-rpm:
        app_label: "rpm"
        source_dir: "git+https://github.com/pulp/pulp_rpm.git@3.0.0b3"
```

If you want to run the tasks of this role in a different host than the
pulp-server, use this variables when installing the server

```yaml
    pulp_api_host: 0.0.0.0
    pulp_content_bind: "0.0.0.0:24816"
```

This permit to access to the API interface from outside the pulp server. Be
sure that the firewall permit the access to the `24816` and `24817` ports.

Requirements
------------

Install this packages in the host where the tasks are executed:

  - httpie
  - jq


Role Variables
--------------

To interact with the pulp server you need to set this variables:

```yaml
    pulp_admin_user: admin
    pulp_default_admin_password: password
    pulp_api_server: http://localhost:24817
```

* `pulp_admin_user`: Is the login name to connect to the API interface. This is
   hard-coded in [pulp_ansible](https://github.com/pulp/pulp_ansible) installer.
   So, this should not change.
* `pulp_default_admin_password`: Is the password to connect to the API
   interface.
* `pulp_api_server`: the pulp server address and port.

### Synchronized repositories

To add and synchronize repositories, add a list of repositories and its remotes:

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

- `repo`: the name of the repository to create.
- `repo_version`: the version of the repository that will be distributed.
- `remote`: the name of the remote you want to synchronize.
- `remote_url`: the address of the remote repository.
- `policy`: **immediate** means that all the content is downloaded right
  away.  **on_demand** the synchronization of a repository is faster and
  downloads RPMs whenever they are requested by clients.
- `base_path`: the path that will be added at the end of the distribution
  address.
- `distribution`: name of the distribution.
- `sync`: synchronize the remote repository when executing the role. This will
  increment the last version even if there has been no modifications in the
  remote repository. It is recommended to set to **yes** at the creation and set
  it to **no** until new packages versions are available in the remote server.
- `state`: if **present**, then the repository and its remote are created or
  nothing happens if they already exists. If **absent**, then the repository and
  its remote are removed. When a repository is removed, all the related
  publications are removed but not the distributions. The role also remove all
  orphans packages not related to a publication.

### Local repositories

To add repositories with your own packages, add a list with repositories and
packages:

```yaml
    pulp_local_repos:
      - repo: repo_a_name
        repo_version: 2
        add_packages:
          - pkg1a-version-release.arch.rpm
          - pkg2a-version-release.arch.rpm
          - pkg3a-version-release.arch.rpm
        remove_packages: []
        pkg_dir: /path1/to/your/packages/
        base_path: base/path
        distribution: dist_a
        state: present
    
      - repo: repo_b_name
        repo_version: latest
        add_packages:
          - pkg1b-version-release.arch.rpm
          - pkg2b-version-release.arch.rpm
          - pkg3b-version-release.arch.rpm
        remove_packages:
          - pkg4b-version-release.arch.rpm
        pkg_dir: /path2/to/your/packages/
        base_path: base/path
        distribution: distri_b
        state: present
```

- `repo`: the name of the repository to create.
- `repo_version`: the version of the repository that will be distributed.
- `packages`: a list of packages that will be added to the repository. If the
  package already exist, nothing happens.
- `pkg_dir`: the pre-path where to find the packages.
- `base_path`: the path that will be added at the end of the distribution
  address. Also the post-path where the role will find the packages. The full
  path will be `{{ pkg_dir }}/{{ base_path }}/{{ package }}`
- `distribution`: name of the distribution.
- `state`: if **present**, then the repository is created or nothing happens if
  it already exists. If **absent**, then the repository and its remote are
  removed. When a repository is removed all the related publications are removed
  but not the distributions. The role also removes all orphans packages not
  related to a publication.


Example Playbook
----------------

Create the following `pulp_api.yml` playbook with the pulp server address and
password:

```yaml
    - hosts: localhost

      roles:
         - { role: pulp_rpm_repos }
```

To sync a repository, create a `sync_repo_foo.yml` file

```yaml
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
```

And execute:

```sh
     ansible-playbook -c local --extra-vars "@sync_repo_foo.yml" playbooks/pulp_api.yml
```

To create a repository and add rpm package, create a `local_repo_a.yml` file

```yaml
    pulp_admin_user: admin
    pulp_default_admin_password: password
    pulp_api_server: http://pulp-server:24817

    pulp_local_repos:
      - repo: repo_a_name
        repo_version: 2
        add_packages:
          - pkg1a-version-release.arch.rpm
          - pkg2a-version-release.arch.rpm
          - pkg3a-version-release.arch.rpm
        remove_packages:
          - pkg4a-version-release.arch.rpm
        pkg_dir: /path1/to/your/packages/
        base_path: base/path
        distribution: dist_a
        state: present
```

And execute:

```sh
     ansible-playbook -c local --extra-vars "@local_repo_a.yml" playbooks/pulp_api.yml
```

License
-------

GPLv2

Author Information
------------------

Juan Cabrera at the [UNamur](https://www.unamur.be/)

