# Autoscale cluster

<!--TOC-->
<!--ENDTOC-->

<!--ROLEVARS-->
## Default variables
```yaml
aws_ec2_autoscale_cluster:
  aws_profile: "{{ _aws_profile }}"
  region: "{{ _aws_region }}"
  name: "example"
  state: present
  acm:
    create_cert: false
    extra_domains: [] # list of Subject Alternative Name domains and zones
    #  - domain: www2.example.com
    #    zone: example.com
    #    aws_profile: us-east-1
    route_53:
      aws_profile: another # the zone might not be in the same account as the certificate
      zone: example.com
  vpc_id: vpc-XXXX # One of vpc_id or vpc_name is mandatory.
  # vpc_name: example-vpc
  subnets:
    # - az: a
    #   cidr: "10.0.3.0/26"
    - az: b
      cidr_block: "10.0.3.64/26"
      # Name of a public subnet in the same AZ.
      public_subnet: public-b
    - az: c
      cidr_block: "10.0.3.128/26"
      public_subnet: public-c
  instance_type: t3.micro
  assign_public_ip: false
  key_name: "{{ ce_provision.username }}@{{ ansible_hostname }}" # This needs to match your "provision" user SSH key.
  ami_owner: self # Default to self-created image.
  root_volume_size: 30
  root_volume_type: gp2 # available options - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html
  root_volume_delete_on_termination: true
  device_name: /dev/xvda
  ebs_optimized: true
  encrypt_boot: false # Whether to encrypt the EBS volumes or not, passed to the aws_ami role and to EBS volumes when instances are built
  ami_playbook_file: "{{ playbook_dir }}/ami.yml"
  ami_operation: create # see aws_ami for details, use 'repack' to build AMIs from running instances, use 'create' to force a new Packer image each time
  # Overrides extra_vars in the aws_ami role.
  ami_extra_vars:
    - "_aws_resource_name: {{ _aws_resource_name }}"
    - "_infra_name: {{ _infra_name }}"
    - "_env_type: {{ _env_type }}"
    - "_aws_region: {{ _aws_region }}"
    - "_aws_profile: {{ _aws_profile }}"
  ami_subnet_name: "{{ _infra_name }}-{{ _env_type }}-a" # Usually cluster subnets do not have Internet Gateways so cannot be used, so we use one that does have an IGW.
  packer_on_error: cleanup # see aws_ami for details
  packer_force: false # see aws_ami for details
  packer_vpc_filter: "" # see aws_ami for details
  packer_subnet_filter_az: "" # see aws_ami for details
  ami_refresh: true # Whether to build a new AMI or not.
  asg_refresh: true # Whether to build a new ASG or not.
  # Define if you want to launch config to use a specific AMI, e.g. to pack a new AMI but not use it right away for QA reasons.
  #image_id: ami-0123456789abcdefg
  asg_cloudwatch_policy_scale_up_name: "{{ _env_type }}-scale-up-policy"
  asg_cloudwatch_policy_scale_down_name: "{{ _env_type }}-scale-down-policy"
  asg_scaling_policies: # List of AutoScale policies. See ec2_scaling_policy module docs for options and details.
    - name: "{{ _env_type }}-scale-up-policy"
      policy_type: "SimpleScaling"
      adjustment_type: "ChangeInCapacity"
      adjustment: 2
      adjustment_step: 1 # Only used when adjustment_type is PercentChangeInCapacity.
      cooldown: 300
    - name: "{{ _env_type }}-scale-down-policy"
      policy_type: "SimpleScaling"
      adjustment_type: "ChangeInCapacity"
      adjustment: -2
      adjustment_step: -1 # Only used when adjustment_type is PercentChangeInCapacity.
      cooldown: 300
  asg_cloudwatch_alarm_scale_up_name: "{{ _env_type }}-cloudwatch-metric-alarm-cpu-scale-up"
  asg_cloudwatch_alarm_scale_down_name: "{{ _env_type }}-cloudwatch-metric-alarm-cpu-scale-down"
  asg_cloudwatch_alarms:
    - scale_direction: "up"
      description: "CPU over 80% so scale up."
      metric: "CPUUtilization"
      namespace: "AWS/EC2"
      statistic: "Average"
      threshold: 80
      unit: "Percent"
      comparison: "GreaterThanOrEqualToThreshold"
      period: 120
      evaluation_periods: 5
    - scale_direction: "down"
      description: "CPU under 40% so scale down."
      metric: "CPUUtilization"
      namespace: "AWS/EC2"
      statistic: "Average"
      threshold: 40
      unit: "Percent"
      comparison: "LessThanOrEqualToThreshold"
      period: 120
      evaluation_periods: 5
  desired_capacity: 0 # Zero means use min_size.
  min_size: 4
  max_size: 8
  # Security groups for the instances cluster.
  # An internal one will be created automatically, use these vars to provide additional groups
  cluster_security_groups: []
  alb_security_groups: []
  efs_security_groups: []
  rds_security_groups: []
  # ALB health checks - these are health check settings applied to the load balancer
  alb_health_check_type: ELB # Uses ALB health checks, set to EC2 to use default AWS instance status checks
  alb_health_check_period: 1200 # Length of time in seconds after a new EC2 instance comes into service that Auto Scaling starts checking its health
  # Target Group health checks - these are distinct and separate checks carried out by the Target Group for the ASG
  deregistration_delay_timeout: 60 # seconds to wait before removing instance from Target Group
  health_check_protocol: "http"
  health_check_path: "/"
  health_check_port: "traffic-port" # Default is traffic-port, which matches the target port. Can be overridden with a port number.
  health_check_success_codes: "200" # Can be single code, like "200", or a comma-separated value with ranges: "200,250-260".
  health_check_interval: 30
  health_check_timeout: 5
  health_check_healthy_count: 5
  health_check_unhealthy_count: 2
  # ALB settings
  create_elb: true # determines whether an ELB (currently, this is an ALB) is created as part of the ASG. This needs to be `true` in order to create a CloudFront distribution.
  alb_idle_timeout: 60
  tags:
    Name: "example"
  # An IAM Role name to associate with instances.
  iam_role_name: "example"
  # Hosts to peer with. This will gather vpc info from the Name tag and create a peering connection and route tables.
  peering:
    - name: utility-server.example.com
      region: "{{ _aws_region }}"
  # Associated RDS instance.
  rds:
    rds: false # wether to create an instance.
    db_instance_class: db.t3.medium
    #db_cluster_identifier: example-aurora-cluster
    engine: mariadb
    aurora_reader: false
    #engine_version: 5.7.9
    # db_parameter_group_name: "example" # Omit to use default
    # db_parameter_group_description: "Custom parameter group" # Description of parameter group
    # db_parameter_group_engine: "mariadb10.5" # accepts different values to RDS instance 'engine'
    # db_parameters: {} # dictionary of available parameters
    allocated_storage: 100 # Initial size in GB. Minimum is 100.
    max_allocated_storage: 1000 # Max size in GB for autoscaling.
    storage_encrypted: false # Whether to encrypt the RDS instance or not.
    master_username: hello # The name of the master user for the DB cluster. Must be 1-16 letters or numbers and begin with a letter.
    master_user_password: hellothere
    multi_az: true
    rds_cloudwatch_alarms:
      - name: "example_free_storage_space_threshold_{{ _env_type }}_asg"
        description: "Average database free storage space over the last 10 minutes too low."
        metric: "FreeStorageSpace"
        namespace: "AWS/RDS"
        statistic: "Average"
        threshold: 20000000000
        unit: "Bytes"
        comparison: "LessThanOrEqualToThreshold"
        period: 600
        evaluation_periods: 1
      - name: "example_cpu_utilization_too_high_{{ _env_type }}_asg"
        description: "Average database CPU utilization over last 10 minutes too high."
        metric: "CPUUtilization"
        namespace: "AWS/RDS"
        statistic: "Average"
        threshold: 65
        unit: "Percent"
        comparison: "GreaterThanOrEqualToThreshold"
        period: 600
        evaluation_periods: 1
      - name: "example_freeable_memory_too_low_{{ _env_type }}_asg"
        description: "Average database freeable memory over last 10 minutes too low, performance may suffer."
        metric: "FreeableMemory"
        namespace: "AWS/RDS"
        statistic: "Average"
        threshold: 100000000
        unit: "Bytes"
        comparison: "LessThanThreshold"
        period: 600
        evaluation_periods: 1
      - name: "example_disk_queue_depth_too_high_{{ _env_type }}_asg"
        description: "Average database disk queue depth over last 10 minutes too high, performance may suffer."
        metric: "DiskQueueDepth"
        namespace: "AWS/RDS"
        statistic: "Average"
        threshold: 64
        unit: "Count"
        comparison: "GreaterThanThreshold"
        period: 600
        evaluation_periods: 1
      - name: "example_swap_usage_too_high_{{ _env_type }}_asg"
        description: "Average database swap usage over last 10 minutes too high, performance may suffer."
        metric: "SwapUsage"
        namespace: "AWS/RDS"
        statistic: "Average"
        threshold: 256000000
        unit: "Bytes"
        comparison: "GreaterThanThreshold"
        period: 600
        evaluation_periods: 1
    sns:
      sns: true
      name: "Notify-Email"
      display_name: "" # Display name for the topic, for when the topic is owned by this AWS account.
      delivery_policy_default_healthy_retry_policy_min_delay_target: 20
      delivery_policy_default_healthy_retry_policy_max_delay_target: 20
      delivery_policy_default_healthy_retry_policy_num_retries: 3
      delivery_policy_default_healthy_retry_policy_num_max_delay_retries: 0
      delivery_policy_default_healthy_retry_policy_num_no_delay_retries: 0
      delivery_policy_default_healthy_retry_policy_num_min_delay_retries: 0
      delivery_policy_default_healthy_retry_policy_backoff_function: "linear"
      delivery_policy_disable_subscription_overrides: false
      subscriptions:
        - endpoint: "admin@example.com"
          protocol: "email"
    backup: "{{ _infra_name }}-{{ _env_type }}" # Whether to back up the RDS instance. Set to the plan you want to use, or an empty string if you don't want to back up.
  # Add an CNAM record tied to the ALB.
  # Set the zone to empty to skip.
  route_53:
    zone: "example.com"
    record: "*.{{ _domain_name }}"
    aws_profile: another # Not necessarily the same as the "target" one.
  ssl_certificate_ARN: ""
  ssl_extra_certificate_ARNs: [] # Optional list of extra certificate ARNs to add to the ALB.
  cloudfront:
    create_cert: false
    create_distribution: false
    cf_certificate_ARN: "" # Certificate must be in us-east-1 for CloudFront. Define a certificate to build a distribution.
  # Add custom listeners. See https://docs.ansible.com/ansible/latest/collections/community/aws/elb_application_lb_module.html
  listeners: []
  alb_ssl_policy: "ELBSecurityPolicy-TLS-1-2-2017-01" # Sets the ALB SSL policy to only accect TLSv1.2 and apply more secure ciphers.
  efs: true # Whether to create an EFS volume.
  efs_encrypt: false # Whether to encrypt the EFS volume
  efs_backup: "{{ _infra_name }}-{{ _env_type }}" # Whether to back up the EFS volume. Set to the plan you want to use, or an empty string if you don't want to back up.

```

<!--ENDROLEVARS-->
