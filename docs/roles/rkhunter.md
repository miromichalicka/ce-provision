# rkhunter
Installs and configures the rkhunter malware scanner.

<!--TOC-->
<!--ENDTOC-->

<!--ROLEVARS-->
## Default variables
```yaml
---
rkhunter:
  apt_autogen: "yes" # automatically update the rkhunter database when apt has run - set to "false" to suppress
  report_email_recipient: system@example.com
  warning_email_recipient: ""
  mail_command: 'mail -s "[rkhunter] Warnings found for ${HOST_NAME}"'
  web_command: "curl"
  bin_directory: "/bin /usr/bin /sbin /usr/sbin /usr/local/bin /usr/local/sbin /usr/libexec /usr/local/libexec"
  log_file: /var/log/rkhunter.log
  append_log: "0"
  copy_log_on_error: "0"
  use_syslog: authpriv.warning
  allow_ssh_root_user: "no"
  disable_tests: "suspscan hidden_procs deleted_files packet_cap_apps apps os_specific"
  scriptwhitelist:
    - /bin/egrep
    - /bin/fgrep
    - /bin/which
    - /usr/bin/ldd
    # - /usr/bin/lwp-request
    - /usr/sbin/adduser
    # - /usr/sbin/prelink
    - /usr/sbin/unhide.rb
  allowhiddendir:
    - /etc/.java
  allowhiddenfile:
    - /etc/.etckeeper
  allowdevfile:
    - /dev/shm/network/ifstate
  allow_system_remote_logging: "0"
  supscan_directories: "/tmp /var/tmp"
  supscan_maxsize: "10240000"
  supscan_threshold: "200"
  use_locking: "0"
  lock_timeout: "300"
  show_lock_messages: "1"
```

<!--ENDROLEVARS-->
