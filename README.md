elb-application-lb
=================

Description
------------

Setup aws application load balancer

Requirements
------------

Related role `elb_target_groups` which will create the target group for the loadbalancer. 


Role Variables
--------------

* `certs` - required - a dict to describe the certificate to be used. 

  It must have field `name` having the value of `domain_name` or `CN` property
  of the cert, and may have field `type` (acm|iam - showing the cert type in
  aws, defauled to 'acm' if not set)

  The assumption is that the certificate has been already imported into aws or
  created there. Use the role acm_cert or iam_cert to upload/update certs into
  aws. For new work, please use acm rather than iam.


* loadbalancer_ related variables. They are all required. See example below.

  Examples:
  ```
loadbalancers:
  - name: "{{env}}-webserver-lb"
    dns_record: webserver-lb # used to crate the route53 iprivate DNS entry for the loadbalancer
    public: True
    access_logs_enabled: yes
    access_logs_s3_location: "{{ logging_bucket_name }}"
    access_logs_s3_prefix: "lb/{{ env }}/webserver"
    target_groups: "{{ target_groups }}"

target_groups:
  new: "{{ target_group_name_new }}"
  old: "{{ target_group_name_old }}"
  default: "{{ target_group_name_default }}"

loadbalancer_listeners:
  - Protocol: "{{ loadbalancer_target_group.protocol|upper }}"
    Port: "{{ loadbalancer_target_group.port }}"
    Certificates: # The whole section can be commented out if you create non ssl LB and no need for certs
      # certificate_arn is populated when elb_application_lb role is run
      # based on the value of certs.name which is the ACM cert domain_name
      - CertificateArn: "{{ certificate_arn | default() }}"
    SslPolicy: ELBSecurityPolicy-2016-08

    DefaultActions:
      - Type: forward
        TargetGroupName: "{{ target_group_name_default }}"
    Rules: #  Can be empty list if u do not need any rules
      - Conditions:
          - Field: host-header
            Values:
              - "new-*.{{ tld_name_external }}"
        Actions:
          - TargetGroupName: "{{ env }}-{{ role_type }}-new-tg"
            Type: forward
        Priority: 1
      - Conditions:
          - Field: host-header
            Values:
              - "old-*.{{ tld_name_external }}"
        Actions:
          - TargetGroupName: "{{ env }}-{{ role_type }}-old-tg"
            Type: forward
        Priority: 2

  ```

For now this role relates to the role elb_target_groups to create TGs. 

These variables are used by that role

```
loadbalancer_health_check:
  protocol: http
  port: 80
  path: "/ping"
  response_timeout: 7
  interval: 30
  unhealthy_threshold: 5
  healthy_threshold: 2

loadbalancer_target_group_protocol: https
loadbalancer_target_group_port: 443
loadbalancer_target_group_connection_draining: 3600

```

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - role: xvt-system-common
           system_common_ntp_service: chrony

License
-------

BSD

Author Information
------------------

XVT DevOps
