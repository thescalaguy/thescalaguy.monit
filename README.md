thescalaguy.monit
=========

A work-in-progress role to simplify setting up monitrc agent on your servers.  
Installs the agent by downloading the monit archive, extracting it, and creating a symlink to the binary.  
Provides support for Slack and email alerting.

Requirements
------------

None

Role Variables
--------------

The variables in `vars` directory are intended to be unchanged. Here's what the `defaults` mean:  

**Variables to control version of monitrc**

| Variable | Meaning |
--- | --- | ---
major | The major version of monitrc agent |
minor | The minor version of monitrc agent |
patch | The patch version of monitrc agent | 

**Variables to enable web UI**

| Variable | Meaning |
--- | --- | ---
enable_httpd | Controls whether we want to enable HTTPD. This enables logging into the web UI on each node. Defaults to true. |
monit_admin_user | The username of the admin user. Defaults to "admin". |
monit_admin_pass | The htpasswd-formatted password for the admin user. Defaults to "monit". You may use [this tool](http://www.htaccesstools.com/htpasswd-generator/) to generate a new password. It is recommended that you change the admin password. |
httpd_port | The port on which HTTPD would be listening. Defaults to "2812". |
use_address | The IP address to which HTTPD would be bound. Defaults to "localhost". You may need to change this to the public IP of the node. |
allow_hosts | A list of hosts who are allowed to connect to this node's agent. Defaults to "0.0.0.0/0.0.0.0". You may need to change this to a list of public IPs who are allowed to connect. |
allow_users | A list of users who are allowed to log into the web UI. By default the admin user is allowed to login. Each entry in this list is added to the htpasswd file. |  

**Variables to configure SMTP**  

| Variable | Meaning |
--- | --- | ---
mail_server_enabled | Controls whether you want to enable SMTP to send alerts via email. |
mail_server_address | The address of your SMTP server. Defaults to "smtp.gmail.com". The default is a good choice if you want a quick setup or else you will need to use your own. |
mail_server_port | The port on which your SMTP server is running. Defaults to "587". |
mail_server_username | The username of your SMTP user who will be sending the email. Defaults to "user". |
mail_server_password | The password of your SMTP user who will be sending the email. Defaults to "password". |  

**Variables to configure Slack integration** 
[Please follow the instructions on monitrc's website](https://mmonit.com/wiki/MMonit/SlackNotification). Those instructions will also explain why Slack variables are named the way they are.  

| Variable | Meaning |
--- | --- | ---
slack_enabled | Controls whether Slack integration is turned on or not. Defaults to "false". |
slack_x | First part of the Slack webhook integration URL. |
slack_y | Second part of the Slack webhook integration URL. |
slack_z | Third part of the Slack webhook integration URL. |  

**Variables to control which websites to monitor** 

| Variable | Meaning |
--- | --- | ---
websites | A list of websites to be monitored. Each element of the list is a dictionary. An example is provided in defaults. The explanation of each key in the dictionary is provided next. |
display_name | How the website will show up in the web UI. |
alert_type | How to send an alert. May be one of "slack" or "mail". In case none is specified, it defaults to "mail". |
cycles | For how many cycles should the website be monitored before raising an alert. This helps in avoiding false positives. |
address | The address of the website. |
ssl | Whether the website has SSL enabled. If "true", you will be notified when your certificate is about to expire. |
alert | A list of email addresses to alert. This is required only when "alert_type" is set to "mail". |  

**Variables to control which filesystems to monitor**  

| Variable | Meaning |
--- | --- | ---  
filesystems | A list of filesystems to monitor. Each element of the list is a dictionary. An example is provided in defaults. The explanation of each key in the dictionary is provided next. |
display_name | How the file system will show up in the web UI. |
alert_type | How to send an alert. May be one of "slack" or "mail". In case none is specified, it defaults to "mail". |
cycles | For how many cycles should the file system be monitored before raising an alert. This helps in avoiding false positives. | 
path | The path to the file system you want to monitor. | 
space_usage | How much the file system space should be used before raising an alert, specified as a percentage. |
service_time | The acceptable service time, in milliseconds, for a file system access beyond which an alert should be raised. |
alert | A list of email addresses to alert. This is required only when "alert_type" is set to "mail". |  


Dependencies
------------

None

Example Playbook
----------------
```
- name: Install Monit
  hosts: monit
  become: true
  vars: 
    listen: "{{ hostvars[inventory_hostname].vagrant_private_ip }}"
  roles: 
    - role: thescalaguy.monit
      use_address: "{{ listen }}"
      mail_server_username: "{{ some_mail_user }}"
      mail_server_password: "{{ mail_server_password  }}" # Use Ansible Vault. Always.
      
      slack_enabled: true
      slack_x: "{{ slack_x }}"
      slack_y: "{{ slack_y }}"
      slaxk_z: "{{ slack_z }}"

      # Out-of-the-box monitoring for MySQL. More to come.
      monitor_mysql: false
      mysql_alert_type: "slack"
      mysql_cycles: 5
      
      # Set to an empty list if you don't want to monitor websites
      websites: 
          - display_name: Google
            alert_type: "slack"
            cycles: 2
            timeout: 60
            address: "google.com"
            ssl: true
            alert:
              - admin@yourcompany.com

      # Set to an empty list if you don't want to monitor disks
      filesystems:
          - display_name: disk1
            alert_type: "mail"
            cycles: 10
            path: /dev/sda1
            space_usage: 95
            service_time: 300
            alert:
              - admin@yourcompany.com
```


License
-------

MIT

Roadmap
-------
1. Add support for monitoring custom processes
2. Add support for monitoring files
3. Add support for monitoring directories
4. Add support for monitoring hosts
5. Add support for monitoring system
6. Add support for monitoring programs
7. Add support for monitoring network
8. Provide more drop-in templates to monitor common processes like MySQL, Postgres, etc.
9. Provide support for running HTTPD with SSL support

Author Information
------------------

fasihxkhatib@gmail.com
