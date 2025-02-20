# Gitlab

<!--TOC-->
<!--ENDTOC-->

## Configuration

Because of the size of the gitlab.rb file, it is impractical to try to parameterized it.
Only a few basic variables are thus provided. To further customize it, the recommended approach is to leverage the "override" system and provide a custom template.

<!--ROLEVARS-->
## Default variables
```yaml
---
# See https://github.com/ansible/ansible/issues/8603

ldap_client:
  endpoints: [""]
  lookup_base: ""
  binddn: ""
  bindpw: ""

gitlab:
  apt_origin: "origin=packages.gitlab.com/gitlab/gitlab-ce,codename=${distro_codename},label=gitlab-ce" # used by apt_unattended_upgrades
  server_name: "gitlab.{{ _domain_name }}"
  # Add a record for GitLab in AWS Route 53
  # If you use the aws_ec2_with_eip role to create your server this will not be necessary
  gitlab_route_53:
    state: present
    zone: "" # empty zone skips DNS creation
    record: "{{ _domain_name }}"
    type: A # change to CNAME if required
    value: 1.2.3.4 # set IP if type: A and target hostname if type: CNAME
    aws_profile: another # Not necessarily the same as the "target" one for the server
    wildcard: true # Creates a matching wildcard CNAME  letsencrypt: "true" # use built-in GitLab LetsEncrypt support by default
  letsencrypt: "true" # GitLab's built in SSL handling enabled by default
  ssl: # @see the 'ssl' role. Note that domain is autopopulated from server_name above.
    enabled: false # manual SSL handling disabled by default
    handling: selfsigned
  # Linux setup
  linux_user: git
  linux_group: git
  linux_uid: nil
  linux_gid: nil
  linux_shell: /bin/sh
  linux_user_home: /var/opt/gitlab
  username: GitLab
  email: "gitlab@#{node['fqdn']}"
  # GitLab settings
  default_theme: 1 # see 'Color theme' under https://gitlab.example.com/-/profile/preferences for options
  disable_signup: true
  disable_signin: false
  private_projects: true
  unicorn_worker_processes: 2
  puma_worker_processes: 2
  initial_root_password: "Ch@ng3m3"
  # LDAP settings
  ldap: false # enable/disable LDAP integration
  ldap_endpoint: "{{ ldap_client.endpoints[0] }}"
  ldap_lookup_base: "{{ ldap_client.lookup_base }}"
  ldap_binddn: "{{ ldap_client.binddn }}"
  ldap_bindpw: "{{ ldap_client.bindpw }}"
  # Mattermost chat settings
  mattermost: false # enable/disable Mattermost chat
  mattermost_url: "chat.{{ _domain_name }}" # unless you use Route 53 integration you must create a DNS record first for LetsEncrypt to work
  # See the Mattermost docs for possibilities, most Mattermost config options have an environment variable version:
  # https://docs.mattermost.com/guides/administration.html#get-started
  mattermost_env_vars: [] # list of environment variables to pass to Mattermost, for example:
  #  - "'MM_EMAILSETTINGS_ENABLESIGNUPWITHEMAIL' => 'false'"
  #  - "'MM_ALLOW_UNTRUSTED_INTERNAL_CONNECTIONS_TO' => 'git.example.com'"
  # Add a CNAME record for Mattermost in AWS Route 53
  mattermost_route_53:
    state: present
    zone: "" # empty zone skips DNS creation
    aws_profile: another # Not necessarily the same as the "target" one for the server
    wildcard: false # Creates a matching wildcard CNAME
  # Single sign-on settings
  omniauth: false # enable/disable SAML logins via Omniauth
  omniauth_auto_link_saml_user: "false"
  omniauth_block_auto_created_users: "true"
  omniauth_login_button_label: "Login with SAML"
  omniauth_consumer_service_url: "https://{{ _domain_name }}/users/auth/saml/callback"
  omniauth_saml_cert_fingerprint: "00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00" # fingerprint of the SAML server's certificate
  omniauth_saml_endpoint_url: https://login.example.com/simplesaml/saml2/idp/SSOService.php # typical endpoint if you followed the SimpleSAMLphp QuickStart - https://simplesamlphp.org/docs/stable/simplesamlphp-idp.html
  omniauth_saml_entity_id: "{{ _domain_name }}" # can be any string, typically just the domain name
  omniauth_saml_attribute_statements: "uid: ['uid']" # typical basic set-up if your SAML authsource is OpenLDAP
  # Other services
  prometheus: "true" # enable/disable built-in Prometheus
  node_exporter: "true" # enable/disable built-in Prometheus Node Exporter
  alertmanager: "true" # enable/disable built-in Prometheus Alertmanager
  nginx:
    enable: true
    listen_port: 443
    listen_https: 443
    client_max_body_size: "250m"
    redirect_http_to_https: "true" # must be enabled if you're using LetsEncrypt above
    redirect_http_to_https_port: 80 # must be 80 if you're using LetsEncrypt above
    custom_nginx_config: "" # include extra config, for example "include /etc/nginx/conf.d/example.conf;"

```

<!--ENDROLEVARS-->
