# Python Pip
Role to install pip3.

<!--TOC-->
<!--ENDTOC-->

<!--ROLEVARS-->
## Default variables
```yaml
---
python_pip:
  python_binary_path: "/usr/bin/python3"
  pip_packages:
    - "python-pip"
    - "python3-pip"
  upgrade:
    enabled: true # create cron job to auto-upgrade pip
    command: "/usr/bin/python -m pip install --upgrade pip"
    # cron variables - see https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html
    minute: 0
    hour: 1
    # day: 1
    # weekday: 7
    # month: 12
    # disabled: true

```

<!--ENDROLEVARS-->
