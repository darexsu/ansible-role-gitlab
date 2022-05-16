# Ansible role GitLab 

[![CI Molecule](https://github.com/darexsu/ansible-role-gitlab/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-gitlab/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58157?color=blue&label=downloads)

  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [merge behaviour](#merge-behaviour)
  - Playbooks (merge version):
      - [install and configure: GitLab, FirewallD](#install-and-configure-gitlab-firewalld-merge-version)
        - [install: GitLab](#install-gitlab-merge-version)
        - [configure: GitLab](#configure-gitlab-merge-version)
        - [configure: GitLab, with self-signed Certificate](#configure-gitlab-with-self-signed-certificate-merge-version)  
  - Playbooks (full version):
      - [install and configure: GitLab, FirewallD](#install-and-configure-gitlab-firewalld-full-version)
        - [install: GitLab](#install-gitlab-full-version)
        - [configure: GitLab](#configure-gitlab-full-version)
        - [configure: GitLab, with self-signed Certificate](#configure-gitlab-with-self-signed-certificate-full-version)  

### Platforms

|  Testing         |  repo: gitlab       |
| :--------------: | :----------------:  |
| Debian 11        |  :heavy_check_mark: |
| Debian 10        |  :heavy_check_mark: |
| Ubuntu 20.04     |  :heavy_check_mark: |
| Ubuntu 18.04     |  :heavy_check_mark: |
| Oracle Linux 8   |  :heavy_check_mark: |
| Rocky Linux 8    |  :heavy_check_mark: |

### Install
```
ansible-galaxy install darexsu.gitlab --force

```
### Requirements

roles: [FirewallD](https://github.com/darexsu/ansible-role-firewalld) (will automatically be installed)

### Merge behaviour
Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```
##### Install and configure: GitLab, FirewallD (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # GitLab
      gitlab:
        enabled: true
        domain: "gitlab.local"
      # GitLab -> install
      gitlab_install:
        enabled: true
      # GitLab -> configure
      gitlab_config:
        enabled: true
        data: |
          external_url 'http://{{ gitlab.domain }}/'
          registry_external_url 'http://{{ gitlab.domain }}'
          letsencrypt['enable'] = false

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        service_http:
          enabled: true
        service_https:
          enabled: true

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Install: GitLab (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:    
      # GitLab
      gitlab:
        enabled: true
        domain: "gitlab.local"
      # GitLab -> install
      gitlab_install:
        enabled: true

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Configure: GitLab (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # GitLab
      gitlab:
        enabled: true
        domain: "gitlab.local"
      # GitLab -> configure
      gitlab_config:
        enabled: true
        data: |
          external_url 'http://{{ gitlab.domain }}/'
          registry_external_url 'http://{{ gitlab.domain }}'
          letsencrypt['enable'] = false

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Configure: GitLab, with self-signed Certificate (merge version)
```yaml
---
- hosts: all
  become: true

  vars:
    merge:
      # GitLab
      gitlab:
        enabled: true
        domain: "gitlab.local"
      # GitLab -> self-signed Certificate Authority
      gitlab_ca:
        enabled: true
        path: "/etc/gitlab/ssl/"
        common_name: "{{ gitlab.domain }}"
        country_name: "US"
        state_or_province_name: "New York"
        locality_name: "New York City"
        organization_name: "Organization_Name"
        organizational_unit_name: "Community"
        email_address: "example@gmail.com"
        basic_constraints: 'CA:TRUE'
        subject_alt_name: "DNS:{{ gitlab.domain }}"
      # GitLab -> configure
      gitlab_config:
        enabled: true
        data: |
          external_url 'https://{{ gitlab.domain }}/'
          registry_external_url 'https://{{ gitlab.domain }}'
          letsencrypt['enable'] = false

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Install and configure: GitLab, FirewallD (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # GitLab
    gitlab:
      enabled: true
      domain: "gitlab.local"
      repo: "gitlab"
      edition: "gitlab-ce"
    # GitLab -> install
    gitlab_install:
      enabled: true
    # GitLab -> configure
    gitlab_config:
      enabled: true
      file: "gitlab.rb"
      src: "gitlab.rb.j2"
      backup: true
      data: |
        external_url 'http://{{ gitlab.domain }}/'
        registry_external_url 'http://{{ gitlab.domain }}'
        letsencrypt['enable'] = false

    # FirewallD
    firewalld:
      enabled: true
      service:
        enabled: true
        state: "started"
    # FirewallD -> rules
    firewalld_rules:
      service_http:
        enabled: true
        zone: "public"
        state: "enabled"
        service: "http"
        permanent: true
      service_https:
        enabled: true
        zone: "public"
        state: "enabled"
        service: "https"
        permanent: true

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Install: GitLab (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # GitLab
    gitlab:
      enabled: true
      domain: "gitlab.local"
      repo: "gitlab"
      edition: "gitlab-ce"
    # GitLab -> install
    gitlab_install:
      enabled: true

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Configure: GitLab (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # GitLab
    gitlab:
      enabled: true
      domain: "gitlab.local"
      repo: "gitlab"
      edition: "gitlab-ce"
    # GitLab -> configure
    gitlab_config:
      enabled: true
      file: "gitlab.rb"
      src: "gitlab.rb.j2"
      backup: true
      data: |
        external_url 'http://{{ gitlab.domain }}/'
        registry_external_url 'http://{{ gitlab.domain }}'
        letsencrypt['enable'] = false

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```
##### Configure: GitLab, with self-signed Certificate (full version)
```yaml
---
- hosts: all
  become: true

  vars:
    # GitLab
    gitlab:
      enabled: true
      domain: "gitlab.local"
      repo: "gitlab"
      edition: "gitlab-ce"
    # GitLab -> self-signed Certificate Authority
    gitlab_ca:
      enabled: true
      path: "/etc/gitlab/ssl/"
      common_name: "{{ gitlab.domain }}"
      country_name: "US"
      state_or_province_name: "New York"
      locality_name: "New York City"
      organization_name: "Organization_Name"
      organizational_unit_name: "Community"
      email_address: "example@gmail.com"
      basic_constraints: 'CA:TRUE'
      subject_alt_name: "DNS:{{ gitlab.domain }}"
    # GitLab -> configure
    gitlab_config:
      enabled: true
      file: "gitlab.rb"
      src: "gitlab.rb.j2"
      backup: false
      data: |
        external_url 'https://{{ gitlab.domain }}/'
        registry_external_url 'https://{{ gitlab.domain }}'
        letsencrypt['enable'] = false

  tasks:
    - name: role darexsu gitlab
      include_role:
        name: darexsu.gitlab
```