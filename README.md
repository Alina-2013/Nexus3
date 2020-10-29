# Nexus3

Nexus Repository is a popular repository manager that provides a single source of truth for all of the software components used by the applications in an enterprise. It provides a single access and control point for Maven (Java(tm)), npm (Node JS) and other software dependencies. It can also be used to manage or govern dependencies, or reduce the build dependencies on multiple external repositories and internet access.

With an application stack, the owner of each stack can provide a default repository manager, force the use of a specific repository manager instance, or block the use of a repository manager. The current Java Microprofile and Node JS Express stacks restrict repository access, so if you want to use Nexus Repository you'll need to create your own stack. You can start by copying and customizing an existing stack rather than starting from scratch.

The remainder of this guide will walk you through the steps required to configure a Nexus Repository for both a Java and Node JS stack. If you are not familiar with customizing stacks, there is a Working with Stacks guide that provides some background and walks you through all of the required steps. If you're already familiar with customizing stacks and have your own, you may want to skip to steps 4 & 5 (for Maven) and steps 2-5 (for NPM) to focus just on the Nexus Repository configuration. Following these steps will create an application stack that uses Nexus Repository management for both development and build time of your applications.


Ansible Role: Nexus 3 OSS

This role installs and configures Nexus Repository Manager OSS version 3.x on CentOS/RHEL.

All configuration can be updated by re-running the role, except for the blobstores-related settings, which are immutable in nexus.

Requirements

This has only been tested on CentOS 7
Oracle Java 8 (mandatory)
Apache HTTPD (optional, used to setup a SSL reverse-proxy)
(see Dependencies section below for matching roles on galaxy)

Role Variables

Ansible variables, along with the default values (see default/main.yml) :

nexus_version: '3.1.0-04'
nexus_timezone: 'UTC'
nexus_package: "nexus-{{ nexus_version }}-unix.tar.gz"
The nexus version and package to install, see available versions at https://www.sonatype.com/download-oss-sonatype . nexus_timezone is a Java Timezone name and can be useful in combination with nexus_scheduled_tasks cron expressions below.

nexus_download_dir: '/opt'
Directory on target where the nexus package will be downloaded.

nexus_default_port: 8081
nexus_default_context_path: '/'
Port and context path of the java nexus process. nexus_default_context_path has to keep the trailing slash when set, for ex. : nexus_default_context_path: '/nexus/'.

nexus_os_group: 'nexus'
nexus_os_user: 'nexus'
User and group used to own the nexus files and run the service, those will be created by the role if absent.

nexus_installation_dir: '/opt'
nexus_data_dir: '/var/nexus'
nexus_tmp_dir: '/opt/nexus'
Nexus directories, nexus_installation_dir contains the installed executable(s), nexus_data_dir contains all configuration, repositories and uploaded artifacts. Note: custom blobstores paths outside of nexus_data_dir can be configured, see nexus_blobstores below.


nexus_anonymous_access: false
Allow anonymous access to nexus.

public_hostname: 'nexus.vm'
The fully qualified domain name under which the nexus instance will be accessible to its clients.

nexus_branding_header: ""
nexus_branding_footer: "Last provisionned {{ ansible_date_time.iso8601 }}"
Header and footer branding, those can contain HTML.

httpd_setup_enable: false
httpd_ssl_certificate_file: 'files/nexus.vm.crt'
httpd_ssl_certificate_key_file: 'files/nexus.vm.key'
Setup an SSL Reverse-proxy, this needs httpd installed. Note : when httpd_setup_enable is set to true, nexus binds to 127.0.0.1:8081 thus not being directly accessible on HTTP port 8081 from an external IP.

ldap_connections: []
LDAP connection(s) setup, each item goes as follow :

  - ldap_name: 'My Company LDAP' # used as a key to update the ldap config
    ldap_protocol: 'ldaps' # ldap or ldaps
    ldap_hostname: 'ldap.mycompany.com'
    ldap_port: 636
    ldap_search_base: 'dc=mycompany,dc=net'
    ldap_auth: 'none' # or simple
    ldap_auth_username: 'username' # if auth = simple
    ldap_auth_password: 'password' # if auth = simple
    ldap_user_base_dn: 'ou=users'
    ldap_user_object_class: 'inetOrgPerson'
    ldap_user_id_attribute: 'uid'
    ldap_user_real_name_attribute: 'cn'
    ldap_user_email_attribute: 'mail'
    ldap_user_subtree: false
    ldap_map_groups_as_roles: false
    ldap_group_base_dn: 'ou=groups'
    ldap_group_object_class: 'posixGroup'
    ldap_group_id_attribute: 'cn'
    ldap_group_member_attribute: 'memberUid'
    ldap_group_member_format: '${username}'
    ldap_group_subtree: false
Example LDAP config for anonymous authentication (anonymous bind), this is also the "minimal" config :

  - ldap_name: 'Simplest LDAP config'
    ldap_protocol: 'ldaps'
    ldap_hostname: 'annuaire.mycompany.com'
    ldap_search_base: 'dc=mycompany,dc=net'
    ldap_port: 636
    ldap_user_id_attribute: 'uid'
    ldap_user_real_name_attribute: 'cn'
    ldap_user_email_attribute: 'mail'
    ldap_user_object_class: 'inetOrgPerson'
Example LDAP config for simple authentication (using a DSA account) :

  - ldap_name: 'LDAP config with DSA'
    ldap_protocol: 'ldaps'
    ldap_hostname: 'annuaire.mycompany.com'
    ldap_port: 636
    ldap_auth: 'simple'
    ldap_auth_username: 'cn=mynexus,ou=dsa,dc=mycompany,dc=net'
    ldap_auth_password: "{{ vault_ldap_dsa_password }}" # better keep passwords in an ansible vault
    ldap_search_base: 'dc=mycompany,dc=net'
    ldap_user_base_dn: 'ou=users'
    ldap_user_object_class: 'inetOrgPerson'
    ldap_user_id_attribute: 'uid'
    ldap_user_real_name_attribute: 'cn'
    ldap_user_email_attribute: 'mail'
    ldap_user_subtree: false
Example LDAP config for simple authentication (using a DSA account) + groups mapped as roles :

  - ldap_name: 'LDAP config with DSA'
    ldap_protocol: 'ldaps'
    ldap_hostname: 'annuaire.mycompany.com'
    ldap_port: 636
    ldap_auth: 'simple'
    ldap_auth_username: 'cn=mynexus,ou=dsa,dc=mycompany,dc=net'
    ldap_auth_password: "{{ vault_ldap_dsa_password }}" # better keep passwords in an ansible vault
    ldap_search_base: 'dc=mycompany,dc=net'
    ldap_user_base_dn: 'ou=users'
    ldap_user_object_class: 'inetOrgPerson'
    ldap_user_id_attribute: 'uid'
    ldap_user_real_name_attribute: 'cn'
    ldap_user_email_attribute: 'mail'
    ldap_map_groups_as_roles: true
    ldap_group_base_dn: 'ou=groups'
    ldap_group_object_class: 'groupOfNames'
    ldap_group_id_attribute: 'cn'
    ldap_group_member_attribute: 'member'
    ldap_group_member_format: 'uid=${username},ou=users,dc=mycompany,dc=net'
    ldap_group_subtree: false
nexus_privileges:
  - name: all-repos-read # used as key to update a privilege
    description: 'Read & Browse access to all repos'
    repository: '*'
    actions: # can be add, browse, create, delete, edit, read or  * (all)
      - read
      - browse
List of the privileges to setup. Those items are combined with the following default values :

    _nexus_privilege_defaults:
      type: repository-view
      format: maven2
      actions:
        - read
    nexus_roles:
      - id: Developpers # can map to a LDAP group id, also used as a key to update a role
        name: developers
        description: All developers
        privileges:
          - nx-search-read
          - all-repos-read
        roles: [] # references to other role names
List of the roles to setup.

nexus_local_users: []
Local (non-LDAP) users/accounts to create in nexus, items go as follow :

  - username: jenkins # used as key to update
    first_name: Jenkins
    last_name: CI
    email: support@company.com
    password: "s3cr3t"
    roles:
      - developers # role ID
nexus_delete_default_repos: false
Delete the repositories from the nexus install initial default configuration. This step is only executed on first-time install (when nexus_data_dir has been detected empty).

nexus_delete_default_blobstore: false
Delete the default blobstore from the nexus install initial default configuration. This can be done only if nexus_delete_default_repos: true and all configured repositories (see below) have an explicit blob_store: custom. This step is only executed on first-time install (when nexus_data_dir has been detected empty).

nexus_blobstores: []
# example blobstore item :
# - name: separate-storage
#   path: /mnt/custom/path
Blobstores to create. A blobstore path and a repository blobstore cannot be updated after initial creation (any update here will be ignored on re-provisionning).

nexus_scheduled_tasks: []
#  example task to compact blobstore :
#  - name: compact-blobstore
#    cron: '0 0 22 * * ?'
#    typeId: blobstore.compact
#    taskProperties:
#      blobstoreName: 'default' # all task attributes are stored as strings by nexus internally
Scheduled tasks to setup. typeId and task-specific taskProperties can be guessed either from the java type hierarchy of org.sonatype.nexus.scheduling.TaskDescriptorSupport or from peeking at the browser AJAX requests while manually configuring a task.

nexus_repos_maven_proxy:
  - name: central
    remote_url: 'https://repo1.maven.org/maven2/'
    layout_policy: permissive
  - name: jboss
    remote_url: 'https://repository.jboss.org/nexus/content/groups/public-jboss/'
# example with a login/password :
# - name: secret-remote-repo
#   remote_url: 'https://company.com/repo/secure/private/go/away'
#   remote_username: 'username'
#   remote_password: 'secret'
Maven proxy repositories configuration.

nexus_repos_maven_hosted:
  - name: private-release
    version_policy: release
    write_policy: allow_once
Maven hosted repositories configuration.

nexus_repos_maven_group:
  - name: public
    member_repos:
      - central
      - jboss
Maven group repositories configuration.

All three repository types are combined with the following default values :

    _nexus_repos_maven_defaults:
      blob_store: default # Note : cannot be updated once the repo has been created
      strict_content_validation: true
      version_policy: release # release, snapshot or mixed
      layout_policy: strict # strict or permissive
      write_policy: allow_once # allow_once or allow
Docker, Pypi, Raw, Rubygems, Bower, NPM repository types: see defaults/main/yml for three options:

  nexus_config_pypi: false
  nexus_config_docker: false
  nexus_config_raw: false
  nexus_config_rubygems: false
  nexus_config_bower: false
  nexus_config_npm: false
These are all false unless you override them from playbook / group_var / cli, these all utlize the same mechanisem as maven.

Dependencies

This role requires Ansible 2.1 or higher.

The java and httpd requirements /can/ be fulfilled with the following galaxy roles :

ansiblebit.oracle-java
geerlingguy.apache
