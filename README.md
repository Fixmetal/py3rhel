py3rhel
=========

This role installs Python 3 on Red Hat Enterprise Linux systems, install pip3 (latest version) with relative dependencies.

Since the coming of RHEL8 (App Stream) the bloatware RHSCL repos are getting obsolete. See [#2353081](https://access.redhat.com/solutions/2353081) for more infos.

The role will install an enabler wrapper script to make Python3 the default (since Python 2.7 is [end of life](https://www.python.org/dev/peps/pep-0373/)).

> :warning: When using Ansible pip module you will need to override the `ansible_python_interpreter` variable overridden because by default the OS will still use Python 2 (hence search for pip2 instance). See examples for this.

Proxy support is intended as Ansible Examples. See [relative page](https://docs.ansible.com/ansible/latest/user_guide/playbooks_environment.html) and playbook examples.

Requirements
------------

Ansible 2.5 for `rhsm_repository` module.

Facts gathering enabled.

A valid subscription to access RHSCL. *OPTIONAL* A valid subscription to access Extras repositories - in case you want `devel` package.

Access to the internet to use pip.

Role Variables
--------------

Available variables are listed below along with default values - see [defaults/main.yml](defaults/main.yml) file:

    py3rhel_delete_pip2: False
    py3rhel_install_devel: False
    py3rhel_python_pkg_name: python3
    py3rhel_python_version: python36
    py3rhel_python_rhscl_pkg_name: rh-{{ py3rhel_python_version }}
    py3rhel_rhscl6_repository: rhel-server-rhscl-6-rpms
    py3rhel_rhscl7_repository: rhel-server-rhscl-7-rpms
    py3rhel_extras_repository: rhel-7-server-extras-rpms
    py3rhel_scl_enabler: /etc/profile.d/enable_rhscl_python.sh

A more prioritized variable is defined in the [vars/main.yml](vars/main.yml) file:

    py3rhel_python_interpreter_script: /usr/local/bin/ansible_python_interpreter.sh


Dependencies
------------

None.

Example Playbook
----------------

Instal Python3:

```yaml
- hosts: all
  vars:
    proxy_env:
      http_proxy: http://proxy.bos.example.com:8080
      https_proxy: http://proxy.bos.example.com:8080
  tasks:
    - name: Install Python3
      become: yes
      import_role:
        name: py3rhel
      environment: "{{ proxy_env | default(omit) }}"
```

Use pip Ansible module to install Wheels on RHEL6/ RHEL7 up to 7.6 (note that the `py3rhel_python_interpreter_script` variable will still be available in the same playbook):

```yaml
    - name: Install Docker and PyYAML Python3 Wheels (RHEL 6.x/RHEL7.x)
      pip:
        name:
          - docker
          - PyYAML
        extra_args: --user
      vars:
        ansible_python_interpreter: "{{ py3rhel_python_interpreter_script }}"
      environment: "{{ proxy_env | default(omit) }}"
      when:
        - ansible_facts['distribution'] == "RedHat"
        - ansible_facts['distribution_major_version']|int == 7
        - ansible_facts['distribution_major_version']|int >= 6
        - ansible_facts['distribution_version'] < "7.7"
```

Use pip Ansible module to install Wheels on RHEL>=7.6 / RHEL 8:

```yaml
    - name: Install Docker and PyYAML Python3 Wheels (RHEL >=7.7)
      pip:
        name:
          - docker
          - PyYAML
        extra_args: --user
      environment: "{{ proxy_env | default(omit) }}"
      vars:
        ansible_python_interpreter: /usr/bin/python3
      when:
        - ansible_facts['distribution'] == "RedHat"
        - ansible_facts['distribution_version'] >= "7.7"
```

License
-------

BSD

Author Information
------------------


