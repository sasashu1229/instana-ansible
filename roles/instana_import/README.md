Note
=========
Change `import_pattern` value in `group_vars/monitor.yaml`

pattern | description
--- | ---
A | Monitor web application consisting of three on-premise servers.<ul><li>DB2</li><li>API (Websphere Liberty)</li><li>WEB (Websphere Liberty)</li></ul>


Set up with GUI
=========

## Create Application
![create new application](./files/img/create_application.png)
![create new application](./files/img/create_application2.png)
![select Services or Endpoints](./files/img/create_application_step1.png)
![add filters](./files/img/create_application_step2.png)

From the running services detected by Instana, select the set of services you wish to define as your application.
![select Service Name](./files/img/create_application_step2_filter.png)
![select service name](./files/img/create_application_step2_filter_service.png)

To monitor all services, connect services with OR condition.
![add OR condition filter](./files/img/create_application_step2_filter_condition.png)

Add other services server2 and sample.
Select `All downstream services`.
![add other filter and select All downstream services](./files/img/create_application_step2_complete.png)

Input any application name.
For example, named `Instana MVP` in this case.
![input application name and confirm create](./files/img/create_application_step3.png)
Press create button.

## Create Smart Alert
Setting about notification to Slack when 5xx error occurs.

![create Smart Alert](./files/img/create_smart_alert.png)
![select HTTP Status Codes](./files/img/create_smart_alert_step1.png)
![select application](./files/img/create_smart_alert_step2.png)

If alert channel is empty, create alert channel.
![select alert channel](./files/img/create_smart_alert_step3.png)
![create alert channel](./files/img/create_smart_alert_step3_channel.png)
![select slack alert](./files/img/create_alert_channel.png)

Label | Value
--- | ---
Name | Alert Channel setting name for example `Instana MVP Slack Alert`
Webhook URL | Slack webhook URL
Icon URL | icon of Slack notification
Channel Name | Slack channel name which receives notification from Instana

![input required fields](./files/img/create_alert_channel_slack.png)
Press Create Alert Channel button.

![select created channel](./files/img/create_smart_alert_step3_after_create_channel.png)
![confirm create](./files/img/create_smart_alert_complete.png)

Create other Smart Alert `Erroneous Calls` by same step.

## Create Custom Events
![create new event](./files/img/create_custom_event.png)
Label | Value
--- | ---
Name | Specify name which indicates what happened
Description | Specify description which indicates what happened
Issue Severity | Critical
Incident | true
Grace Period | 5s

![input and select fields](./files/img/create_custom_event_step1.png)

In this case, set up condition to alert when the free space on disk partition on host drops below 13 GB.
![input and select fields](./files/img/create_custom_event_step2.png)
![select scope](./files/img/create_custom_event_step3.png)

Create these Events.

Event | Condition
--- | ---
Disk Full | Entity type: Host<br>Metric: Filesystem > Free<br>Matching Operator: is<br>Device: /dev/xvda2<br>Time Window: 5s<br>Aggregation: avg<br>Operator: <=<br>Kilobytes: 13631488 (13GB)
Memory Leak | Entity type: JVM<br>Metric: Memory > Used percentage<br>Time Window: 5s<br>Aggregation: avg<br>Operator: >=<br>Percentage: 80
DB Deadlock | Entity type: DB2<br>Metric: Lock Wait<br>Time Window: 5s<br>Aggregation: sum<br>Operator: >=<br>Count: 1

## Create Alert
![create new alert](./files/img/create_alert.png)

Add events created in the previous step.
- Disk Full
- Memory Leak
- DB Deadlock

![input name and select events](./files/img/create_alert_step1.png)
![select scope and alert channel](./files/img/create_alert_step2.png)

Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
