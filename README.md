[![Ansible Galaxy](https://ansible.l3d.space/svg/roles-ansible.forgejo.svg)](https://galaxy.ansible.com/ui/standalone/roles/l3d/forgejo/)
[![BSD-3 Clause](https://ansible.l3d.space/svg/roles-ansible.forgejo_license.svg)](LICENSE)
[![Maintenance](https://ansible.l3d.space/svg/roles-ansible.forgejo_maintainance.svg)](https://ansible.l3d.space/#l3d.forgejo)

 ansible role forgejo
============================

```
STOP

THIS ROLE IS GOING TO CHANGE TO A FORGEJO ONLY ROLE.
SOME VARIABLE NAMES WILL PROBABLY CHANGE
```


This role installs and manages [forgejo](https://forgejo.org). A painless self-hosted Git service. Forgejo is a community managed lightweight code hosting solution written in Go. Forgejo is a hard fork of Gitea.
[Source code forgejo](https://code.forgejo.org/forgejo/forgejo).
This role is also Part of the Ansible-Collection [l3d.git](https://galaxy.ansible.com/l3d/git). [![l3d.git](https://ansible.l3d.space/svg/l3d.git_ansible-collection_collection.svg)](https://github.com/roles-ansible/ansible_collection_git.git).

## Mirrors
The role is mirrored to:
+ Github: [github.com/roles-ansible/ansible_role_forgejo](https://github.com/roles-ansible/ansible_role_forgejo.git)
<!-- + Gitea: [git.l3d.ch/ansible/ansible_role_forgejo](https://git.l3d.ch/ansible/ansible_role_forgejo.git) -->
More about it at [ansible.l3d.space](https://ansible.l3d.space/#l3d.forgejo)

## Sample Usage in a playbook

The following code has been tested with the latest Debian Stable, it should work on Ubuntu and RedHat as well.

```yaml
# ansible-galaxy role install roles-ansible.forgejo

- name: "Install forgejo"
  hosts: git.example.com
  roles:
    - {role: roles-ansible.forgejo, tags: forgejo}
  vars:
    # Here we assume we are behind a reverse proxy that will
    # handle https for us, so we bind on localhost:3000 using HTTP
    # see https://forgejo.org/docs/latest/admin/reverse-proxy/#nginx
    forgejo_fqdn: 'git.example.com'
    forgejo_root_url: 'https://git.example.com'
    forgejo_protocol: http
    forgejo_start_ssh: true
```

## Choosing between Forgejo's built-in SSH and host SSH Server

Forgejo has a built-in SSH server which is running on port 2222 (to not conflict with the host SSH server which usually running on port 22).
This one is used by default in this role and results in a SSH clone URL of `forgejo@<fqdn>:2222:<user>/<repo>.git` because `forgejo` is the default `RUN_AS` user.

Often enough, one wants to have a "clean" SSH URL like `git@<fqdn>:<user>/<repo>.git`.
This is possible by using the host SSH server with the following variable configuration:

```yaml
forgejo_ssh_port: 22 # assuming the host SSH server is running on port 22
forgejo_user: git # otherwise there will be permission issues
forgejo_start_ssh: false # to not start the built-in SSH server
```

The above configuration works out of the box for new installations.
When migrating from a running instance with existing SSH keys from the built-in SSH server to the host SSH server, you need to make sure that the host SSH server is running and that the `forgejo_user` has the necessary permissions to access the repository data and the keys (stored in `<forgejo_home>/.ssh/`)

NB: To use `git@` as described above, `forgejo_user` must be `git` and it does not suffice to set `forgejo_ssh_user: git`.
See [this issue](https://github.com/go-gitea/gitea/issues/28563) for more information..

 Variables
-----------
Here is a deeper insight into the variables of this forgejo role. For the exact function of some variables and the possibility to add more options we recommend a look at this [config cheat sheet](https://forgejo.org/docs/latest/admin/config-cheat-sheet/).

### Forgejo update mechanism
It is advisable to define exactly which Forgejo release you want to install. See [Forgejo releases](https://forgejo.org/releases/) for the correct value to use in `forgejo_version` eg `v1.21.5`.

This is because the Forgejo project maintains both `stable` and `old stable` releases and the `latest` tag will refer to the *most recent release* regardless of whether it is from the `stable` or `old stable` series. This can lead to a situation where `latest` refers to an *older release* than the version you have installed.

### forgejo update
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_version` | `latest` | Define either the exact release to install *(eg. `1.16.0`)* or use ``latest`` *(default)* to install the latest release. |
| `forgejo_version_check` | `true` | Check if installed version != `forgejo_version` before initiating binary download |
| `forgejo_gpg_key` | `EB114F5E6C0DC2BCDD183550A4B61A2DC5923710` | the gpg key the forgejo binary is signed with |
| `forgejo_gpg_server` | `hkps://keys.openpgp.org` | A gpg key server where this role can download the gpg key |
| `forgejo_backup_on_upgrade` | `false` | Optionally a backup can be created with every update of forgejo. |
| `forgejo_backup_location` | `{{ forgejo_home }}/backups/` | Where to store the forgejo backup if one is created with this role. |
| `submodules_versioncheck` | `false` | a simple version check that can prevent you from accidentally running an older version of this role. *(recommended)* |

### forgejo in the linux world
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_group` | `forgejo` | Primary UNIX group used by Forgejo |
| `forgejo_groups` | null | Optionally a list of secondary UNIX groups used by Forgejo |
| `forgejo_home` | `/var/lib/forgejo` | Base directory to work |
| `forgejo_user_home` | `{{ forgejo_home }}` | home of forgejo user |
| `forgejo_executable_path` | `/usr/local/bin/forgejo` | Path for forgejo executable |
| `forgejo_configuration_path` | `/etc/forgejo` | Where to put the forgejo.ini config |
| `forgejo_shell` | `/bin/false` | UNIX shell used by forgejo. Set it to `/bin/bash` if you don't use the forgejo built-in ssh server. |
| `forgejo_systemd_cap_net_bind_service` | `false` | Adds `AmbientCapabilities=CAP_NET_BIND_SERVICE` to systemd service file |

### Overall ([DEFAULT](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#overall-default))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_app_name` | `Forgejo` | Displayed application name |
| `forgejo_user` | `forgejo ` | UNIX user used by Forgejo |
| `forgejo_run_mode`| `prod`| Application run mode, affects performance and debugging. Either “dev”, “prod” or “test”. |
| `forgejo_fqdn` | `localhost` | Base FQDN for the installation, used as default for other variables. Set it to the FQDN where you can reach your forgejo server |

### Repository ([repository](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#repository-repository))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_default_branch` | `main` | Default branch name of all repositories. |
| `forgejo_default_private` | `last` | Default private when creating a new repository. [`last`, `private`, `public`] |
| `forgejo_default_repo_units` | *(see defaults)* | Comma separated list of default repo units. See official docs for more |
| `forgejo_disabled_repo_units` | | Comma separated list of globally disabled repo units. |
| `forgejo_disable_http_git` | `false` | Disable the ability to interact with repositories over the HTTP protocol. (true/false) |
| `forgejo_disable_stars` | `false` | Disable stars feature. |
| `forgejo_enable_push_create_org` | `false` | Allow users to push local repositories to Forgejo and have them automatically created for an org. |
| `forgejo_enable_push_create_user` | `false` | Allow users to push local repositories to Forgejo and have them automatically created for an user. |
| `forgejo_force_private` | `false` | Force every new repository to be private. |
| `forgejo_user_repo_limit` | `-1` | Limit how many repos a user can have *(`-1` for unlimited)* |
| `forgejo_repository_root` | `{{ forgejo_home }}/repos` |  Root path for storing all repository data. It must be an absolute path. |
| `forgejo_repository_extra_config` | | you can use this variable to pass additional config parameters in the `[repository]` section of the config. |

### Repository - Upload ([repository.upload](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#repository---upload-repositoryupload))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_repository_upload_enabled` | `true` | Whether repository file uploads are enabled |
| `forgejo_repository_upload_max_size` | `4` | Max size of each file in megabytes. |
| `forgejo_repository_upload_extra_config` | | you can use this variable to pass additional config parameters in the `[repository.upload]` section of the config. |

### Repository - Signing ([repository.signing](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#repository---signing-repositorysigning))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_enable_repo_signing_options` | `false` | Allow to configure repo signing options |
| `forgejo_repo_signing_key` | `default` | Key to sign with. |
| `forgejo_repo_signing_name` | | if a KEYID is provided as the `forgejo_repo_signing_key`, use these as the Name and Email address of the signer. |
| `forgejo_repo_signing_email` | | if a KEYID is provided as the `forgejo_repo_signing_key`, use these as the Name and Email address of the signer. |
| `forgejo_repo_initial_commit` | `always` | Sign initial commit. |
| `forgejo_repo_default_trust_model` | `collaborator` | The default trust model used for verifying commits. |
| `forgejo_repo_wiki` | `never` |  Sign commits to wiki. |
| `forgejo_repo_crud_actions` | *(see defaults)* | Sign CRUD actions. |
| `forgejo_repo_merges` | *(see defaults)* | Sign merges. |
| `forgejo_enable_repo_signing_extra` | | you can use this variable to pass additional config parameters in the `[repository.signing]` section of the config. |

### CORS ([cors](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#cors-cors))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_enable_cors` | `false` | enable cors headers (disabled by default) |
| `forgejo_cors_scheme` | `http` | scheme of allowed requests |
| `forgejo_cors_allow_domain` | `*` | list of requesting domains that are allowed |
| `forgejo_cors_allow_subdomain` | `false` |allow subdomains of headers listed above to request |
| `forgejo_cors_methods` | *(see defaults)* | list of methods allowed to request |
| `forgejo_cors_max_age` | `10m` | max time to cache response |
| `forgejo_cors_allow_credentials` | `false` | allow request with credentials |
| `forgejo_cors_headers` | `Content-Type,User-Agent` | additional headers that are permitted in requests |
| `forgejo_cors_x_frame_options` | `SAMEORIGIN` |  Set the `X-Frame-Options` header value. |
| `forgejo_cors_extra` | | you can use this variable to pass additional config parameters in the `[cors]` section of the config. |

### UI ([ui](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#ui-ui))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_show_user_email` | `false` | Do you want to display email addresses ? (true/false) |
| `forgejo_theme_default` | `forgejo-auto` | Default theme |
| `forgejo_ui_extra_config` | | you can use this variable to pass additional config parameters in the `[ui]` section of the config. |

### UI - Metadata ([ui.meta](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#ui---metadata-uimeta))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_ui_author` | *(see defaults)* | Author meta tag of the homepage. |
| `forgejo_ui_description` | *(see defaults)* | Description meta tag of the homepage. |
| `forgejo_ui_keywords` | *(see defaults)* | Keywords meta tag of the homepage |
| `forgejo_ui_meta_extra_config` | | you can use this variable to pass additional config parameters in the `[ui.meta]` section of the config. |

### Server ([server](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#server-server))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_protocol`| `http` | Listening protocol [http, https, fcgi, unix, fcgi+unix] |
| `forgejo_http_domain` | `{{ forgejo_fqdn }}` which is `localhost` | Domain name of this server. |
| `forgejo_root_url` | `http://{{ forgejo_fqdn }}:3000` | Root URL used to access your web app (full URL) |
| `forgejo_http_listen` | `127.0.0.1` | HTTP listen address |
| `forgejo_http_port` | `3000` | Bind port *(redirect from `80` will be activated if value is `443`)* |
| `forgejo_start_ssh` | `true` | When enabled, use the built-in SSH server. |
| `forgejo_ssh_domain` | `{{ forgejo_fqdn }} ` |  Domain name of this server, used for displayed clone URL |
| `forgejo_ssh_port` | `2222` | SSH port displayed in clone URL. |
| `forgejo_ssh_listen` | `0.0.0.0` | Listen address for the built-in SSH server. |
| `forgejo_offline_mode` | `true` | Disables use of CDN for static files and Gravatar for profile pictures. (true/false) |
| `forgejo_landing_page` | `home` | Landing page for unauthenticated users |
| `forgejo_lfs_server_enabled` | `false` | Enable GIT-LFS Support *(git large file storage: [git-lfs](https://git-lfs.github.com/))*. |
| `forgejo_lfs_jwt_secret` |  | LFS authentication secret. Can be generated with ``forgejo generate secret JWT_SECRET``. Will be autogenerated if not defined |
| `forgejo_redirect_other_port` | `false` | If true and `forgejo_protocol` is https, allows redirecting http requests on `forgejo_port_to_redirect` to the https port Forgejo listens on. |
| `forgejo_port_to_redirect` | `80` | Port for the http redirection service to listen on, if enabled |
| `forgejo_enable_tls_certs` | `false` | Write TLS Cert and Key Path to config file |
| `forgejo_tls_cert_file` | `https/cert.pem` |  Cert file path used for HTTPS. |
| `forgejo_tls_key_file` | `https/key.pem` | Key file path used for HTTPS. |
| `forgejo_enable_acme` | `false` | Flag to enable automatic certificate management via an ACME capable CA Server. *(default is letsencrypt)* |
| `forgejo_acme_url` | | The CA’s ACME directory URL |
| `forgejo_acme_accepttos` | `false` | This is an explicit check that you accept the terms of service of the ACME provider. |
| `forgejo_acme_directory` | `https` | Directory that the certificate manager will use to cache information such as certs and private keys. |
| `forgejo_acme_email` | | Email used for the ACME registration |
| `forgejo_acme_ca_root` | | The CA’s root certificate. If left empty, it defaults to using the system’s trust chain. |
| `forgejo_server_extra_config` |  | you can use this variable to pass additional config parameters in the `[server]` section of the config. |

### Database ([database](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#database-database))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_db_type` | `sqlite3` | The database type in use `[mysql, postgres, mssql, sqlite3]`. |
| `forgejo_db_host` | `127.0.0.0:3306` | Database host address and port or absolute path for unix socket [mysql, postgres] (ex: `/var/run/mysqld/mysqld.sock`). |
| `forgejo_db_name` | `root` | Database name |
| `forgejo_db_user` | `forgejo` | Database username |
| `forgejo_db_password` | `lel` | Database password. **PLEASE CHANGE** |
| `forgejo_db_ssl` | `disable` | Configure SSL only if your database type supports it. Have a look into the [config-cheat-sheet](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#database-database) for more detailed information |
| `forgejo_db_path` | `{{ forgejo_home }}/data/forgejo.db` | DB path, if you use `sqlite3`. |
| `forgejo_db_log_sql` | `false` | Log the executed SQL. |
| `forgejo_database_extra_config` | | you can use this variable to pass additional config parameters in the `[database]` section of the config. |

### Indexer ([indexer](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#indexer-indexer))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_repo_indexer_enabled` | `false` | Enables code search *(uses a lot of disk space, about 6 times more than the repository size).* |
| `forgejo_repo_indexer_include` |  |Glob patterns to include in the index *(comma-separated list)*. An empty list means include all files. |
| `forgejo_repo_indexer_exclude` |  | Glob patterns to exclude from the index (comma-separated list). |
| `forgejo_repo_exclude_vendored` | `true` | Exclude vendored files from index. |
| `forgejo_repo_indexer_max_file_size` | `1048576` | Maximum size in bytes of files to be indexed. |
| `forgejo_indexer_extra_config` |  | you can use this variable to pass additional config parameters in the `[indexer]` section of the config. |
| `forgejo_queue_issue_indexer_extra_config` | | | you can use this variable to pass additional config parameters in the `[queue.issue_indexer]` section of the config. |

### Security ([security](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#security-security))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_secret_key` | | Global secret key. Will be autogenerated if not defined. Should be unique. |
| `forgejo_disable_git_hooks` | `true` | Set to false to enable users with git hook privilege to create custom git hooks. Can be dangerous. |
| `forgejo_disable_webhooks`  | `false` | Set to true to disable webhooks feature. |
| `forgejo_internal_token` | | Internal API token. Will be autogenerated if not defined. Should be unique. |
| `forgejo_password_check_pwn` | `false` | Check [HaveIBeenPwned](https://haveibeenpwned.com/Passwords) to see if a password has been exposed. |
| `forgejo_security_extra_config` | | you can use this variable to pass additional config parameters in the `[security]` section of the config. |

### Service ([service](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#service-service))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_disable_registration` | `false` | Do you want to disable user registration? (true/false) |
| `forgejo_register_email_confirm` | `false` | Enable this to ask for mail confirmation of registration. Requires `forgejo_mailer_enabled` to be enabled. |
| `forgejo_require_signin` | `true` | Do you require a signin to see repo's (even public ones)? (true/false)|
| `forgejo_default_keep_mail_private` | `true` | By default set users to keep their email address privat |
| `forgejo_enable_captcha` | `true` | Do you want to enable captcha's ? (true/false)|
| `forgejo_show_registration_button` | `true` | Here you can hide the registration button. This will not disable registration! (true/false)|
| `forgejo_only_allow_external_registration` | `false` | Set to true to force registration only using third-party services (true/false) |
| `forgejo_enable_notify_mail` | `false` | Enable this to send e-mail to watchers of a repository when something happens, like creating issues (true/false) |
| `forgejo_auto_watch_new_repos` | `true` | Enable this to let all organisation users watch new repos when they are created (true/false) |
| `forgejo_autowatch_on_change` | `true` | Enable this to make users watch a repository after their first commit to it (true/false) |
| `forgejo_register_manual_confirm` | `false` | Enable this to manually confirm new registrations. Requires REGISTER_EMAIL_CONFIRM to be disabled. |
| `forgejo_default_allow_create_organization` | `false` | Allow new users to create organizations by default (true/false) |
| `forgejo_email_domain_allowlist` | | If non-empty, comma separated list of domain names that can only be used to register on this instance, wildcard is supported. |
| `forgejo_default_user_visibility` | `public` | Set default visibility mode for users, either "public", "limited" or "private". |
| `forgejo_default_org_visibility` | `public` | Set default visibility mode for organisations, either "public", "limited" or "private". |
| `forgejo_allow_only_internal_registration` | `false` | Set to true to force registration only via Forgejo. |
| `forgejo_allow_only_external_registration` | `false` | Set to true to force registration only using third-party services. |
| `forgejo_show_milestones_dashboard_page` | `true` | Enable this to show the milestones dashboard page - a view of all the user's milestones |
| `forgejo_default_user_is_restricted` | `false` | Give new users restricted permissions by default (true/false) |
| `forgejo_service_extra_config` | | you can use this variable to pass additional config parameters in the `[service]` section of the config. |

### Mailer ([mailer](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#mailer-mailer))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_mailer_enabled` | `false` | Whether to enable the mailer. |
| `forgejo_mailer_protocol` | `dummy` |Mail server protocol. One of “smtp”, “smtps”, “smtp+starttls”, “smtp+unix”, “sendmail”, “dummy”.|
| `forgejo_mailer_smtp_addr` | | Mail server address. e.g. smtp.gmail.com. For smtp+unix, this should be a path to a unix socket instead. |
| `forgejo_mailer_smtp_port` | | Mail server port |
| `forgejo_mailer_use_client_cert` | `false` | Use client certificate for TLS/SSL. |
| `forgejo_mailer_client_cert_file` | | Client certificate file. |
| `forgejo_mailer_client_key_file` | | Client key file. |
| `forgejo_mailer_force_trust_server_cert` | `false` | completely ignores server certificate validation errors. This option is unsafe. Consider adding the certificate to the system trust store instead. |
| `forgejo_mailer_user` | | Username of mailing user (usually the sender’s e-mail address). |
| `forgejo_mailer_password ` | |Password of mailing user. Use `your password` for quoting if you use special characters in the password. |
| `forgejo_mailer_enable_helo` | `true` |Enable HELO operation. |
| `forgejo_mailer_from` | `noreply@{{ forgejo_http_domain }}` | Mail from address, RFC 5322. |
| `forgejo_subject_prefix` | |Prefix to be placed before e-mail subject lines. |
| `forgejo_mailer_send_as_plaintext` | `false` | Send mails only in plain text, without HTML alternative. |
| `forgejo_mailer_extra_config` | | you can use this variable to pass additional config parameters in the `[mailer]` section of the config. |

### Session ([session](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#session-session))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_session_provider` | `file` | Session engine provider |
| `forgejo_session_extra_config` | | you can use this variable to pass additional config parameters in the `[session]` section of the config. |

### Picture ([picture](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#picture-picture))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_picture_extra_config` | | you can use this variable to pass additional config parameters in the `[picture]` section of the config. |

### Issue and pull request attachments ([attachment](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#issue-and-pull-request-attachments-attachment))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `attachment_enabled` | `true` | Whether issue and pull request attachments are enabled. |
| `forgejo_attachment_types` | see Docs | Comma-separated list of allowed file extensions (`.zip,.txt`), mime types (`text/plain`) or wildcard type (`image/*`, `audio/*`, `video/*`). Empty value or `*/*` allows all types. |
| `forgejo_attachment_max_size` | `4` | Maximum size (MB). |
| `forgejo_attachment_extra_config` | | you can use this variable to pass additional config parameters in the `[attachment]` section of the config. |

### Log ([log](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#log-log))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_log_systemd` | `false` | Disable logging into `file`, use systemd-journald |
| `forgejo_log_level` | `Warn` | General log level. `[Trace, Debug, Info, Warn, Error, Critical, Fatal, None]` |
| `forgejo_log_extra_config` | | you can use this variable to pass additional config parameters in the `[log]` section of the config. |

### Metrics ([metrics](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#metrics-metrics))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_metrics_enabled`| `false` | Enable the metrics endpoint |
| `forgejo_metrics_token`| | Bearer token for the Prometheus scrape job |
| `forgejo_metrics_extra` | | you can use this variable to pass additional config parameters in the `[metrics]` section of the config. |

### OAuth2 ([oauth2](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#oauth2-oauth2))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_oauth2_enabled` | `true` | Enable the Oauth2 provider (true/false) |
| `forgejo_oauth2_jwt_secret` |  | Oauth2 JWT secret. Can be generated with ``forgejo generate secret JWT_SECRET``. Will be autogenerated if not defined. |
| `forgejo_oauth2_extra_config` |  | you can use this variable to pass additional config parameters in the `[oauth2]` section of the config. |

### Federation ([federation](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#federation-federation))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_federation_enabled` | `false` | Enable/Disable federation capabilities |
| `forgejo_federation_share_user_stats` | `false` | Enable/Disable user statistics for nodeinfo if federation is enabled |
| `forgejo_federation_extra` | | you can use this variable to pass additional config parameters in the `[federation]` section of the config. |

### Packages ([packages](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#packages-packages))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_packages_enabled` | `true` | Enable/Disable package registry capabilities |
| `forgejo_packages_extra` | |you can use this variable to pass additional config parameters in the `[packages]` section of the config. |

### LFS ([lfs](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#lfs-lfs))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_lfs_storage_type` | `local` | Storage type for lfs |
| `forgejo_lfs_serve_direct` | `false` | Allows the storage driver to redirect to authenticated URLs to serve files directly. *(only Minio/S3)* |
| `forgejo_lfs_content_path` | `{{ forgejo_home }}/data/lfs` | Where to store LFS files |
| `forgejo_lfs_extra` | | you can use this variable to pass additional config parameters in the `[lfs]` section of the config. |

### Actions ([actions](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#actions-actions))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_actions_enabled` | `false` | Enable/Disable actions capabilities globally. You may want to add `repo.actions` to `forgejo_default_repo_units` to enable actions on all new repositories |
| `forgejo_actions_default_actions_url` | `github` | Default address to get action plugins, e.g. the default value means downloading from `https://github.com/actions/checkout` for `uses: actions/checkout@v3` |
| `forgejo_actions_extra` | | you can use this variable to pass additional config parameters in the `[actions]` section of the config. |

### Other ([other](https://forgejo.org/docs/latest/admin/config-cheat-sheet/#other-other))
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_other_show_footer_version` | `true` | Show Forgejo and Go version information in the footer. |
| `forgejo_other_show_footer_template_load_time` | `true` | Show time of template execution in the footer. |
| `forgejo_other_enable_sitemap` | `true` | Generate sitemap. |
| `forgejo_other_enable_feed` | `true` | Enable/Disable RSS/Atom feed. |

### additional forgejo config
| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_extra_config` | | Additional forgejo configuration. Have a look at the [config-cheat-sheet](https://forgejo.org/docs/latest/admin/config-cheat-sheet/) before using it! |

### Fail2Ban configuration

If enabled, this will deploy a fail2ban filter and jail config for Forgejo as described in the [Gitea Documentation](https://docs.gitea.io/en-us/fail2ban-setup/) (there is no up to date Forgejo specific documentation for Fail2ban).

As this will only deploy config files, fail2ban already has to be installed or otherwise the role will fail.

| variable name | default value | description |
| ------------- | ------------- | ----------- |
| `forgejo_fail2ban_enabled` | `false` | Whether to deploy the fail2ban config or not |
| `forgejo_fail2ban_jail_maxretry` | `10` | fail2ban jail `maxretry` setting. |
| `forgejo_fail2ban_jail_findtime` | `3600` | fail2ban jail `findtime` setting. |
| `forgejo_fail2ban_jail_bantime` | `900` | fail2ban jail `bantime` setting. |
| `forgejo_fail2ban_jail_action` | `iptables-allports` | fail2ban jail `action` setting. |

### local forgejo Users
| variable | option | description |
| -------- | ------ | ----------- |
| ``forgejo_users`` | | dict to create local forgejo users |
| | ``name`` | name for local forgejo user |
| | ``password`` | user for local git user |
| | ``email`` | email for local git user |
| | ``admin`` | give user admin permissions |
| | ``must_change_password`` | user should change password after first login |
| | ``state`` | set to ``absent`` to delete user |

### optional customisation
You can optionally customize your forgejo using this ansible role. We got our information about customisation from [docs.gitea.io/en-us/customizing-gitea](https://docs.gitea.io/en-us/customizing-gitea/) and [https://forgejo.org/docs/next/contributor/customization/](https://forgejo.org/docs/next/contributor/customization/).
To deploy multiple files we created the ``forgejo_custom_search`` variable, that can point to the path where you put the custom forgejo files *( default ``"files/host_files/{{ inventory_hostname }}/forgejo"``)*.

+ **LOGO**:
  - Set ``forgejo_customize_logo`` to ``true``
  - We search for:
    * ``logo.svg`` - Used for favicon, site icon, app icon
    * ``logo.png`` - Used for Open Graph
    * ``favicon.png`` - Used as fallback for browsers that don’t support SVG favicons
    * ``apple-touch-icon.png`` - Used on iOS devices for bookmarks
  - We search in *(using [first_found](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/first_found_lookup.html))*:
    * ``{{ forgejo_custom_search }}/forgejo_logo/``
    * ``files/{{ inventory_hostname }}/forgejo_logo/``
    * ``files/{{ forgejo_http_domain }}/forgejo_logo/``
    * ``files/forgejo_logo/``
+ **FOOTER**:
  - Set ``forgejo_customize_footer`` to ``true``
  - We Search using first_found in:
    * "{{ forgejo_custom_search }}/forgejo_footer/extra_links_footer.tmpl"
    * "files/{{ inventory_hostname }}/forgejo_footer/extra_links_footer.tmpl"
    * "files/{{ forgejo_http_domain }}/forgejo_footer/extra_links_footer.tmpl"
    * 'files/forgejo_footer/extra_links_footer.tmpl'
    * 'files/extra_links_footer.tmpl'
+ **CUSTOM FILES**:
  - Set ``forgejo_customize_files`` to ``true``
  - Create a directory with the files you want to deploy.
  - Point ``forgejo_customize_files_path`` to this directory. *(Default ``{{ forgejo_custom_search }}/forgejo_files/``)*
+ **CUSTOM THEMES**:
  - Set `forgejo_custom_themes` to a list with URLs for custom theme CSS files. You usually want three individual files per theme. Example:
    ```yaml
    forgejo_custom_themes:
      - https://example.com/theme-custom-auto.css
      - https://example.com/theme-custom-dark.css
      - https://example.com/theme-custom-light.css
    ```
  - Set `forgejo_themes` variable and include the names of the new themes. To keep the existing ones, you need to pass all themes names, e.g. `forgejo-auto,<custom-auto>,<custom-light>,<custom-dark>`

## Requirements
This role uses the ``ansible.builtin`` and ``community.general`` ansible Collections. To download the latest forgejo release we use json_query. This requires ``jmespath`` to be available.

### Python packages
+ jmespath

### Galaxy Collections
+ community.general

### Example requirements Installation
```
ansible-galaxy collection install --update --role-file requirements.yml
pip3 install --update jmespath
```

## Contribute
Don't hesitate to create a pull request, and if in doubt you can reach me at
Mastodon [@l3d@chaos.social](https://chaos.social/@l3d).

I'll be happy to fix any issues you raise, or even better, review your pull requests :)

## History of this role
This ansible role was originally developed on [github.com/thomas-maurice/ansible-role-gitea](https://github.com/thomas-maurice/ansible-role-gitea.git). Since the role there had some problems like default values for the location of the gitea repositories and the merging of pull requests usually takes several months, a fork of the role was created that offered the same. Only tidier and with the claim to react faster to issues and pull requests.

Later the role was modified to allow the installation of Forgejo or Gitea. Following the divergence of Forgejo from Gitea in 2024, this role was forked into specific Gitea and Forgejo versions at the beginning of 2025.

It is now Part of the [l3d.git](https://galaxy.ansible.com/l3d/git) Collection too.
