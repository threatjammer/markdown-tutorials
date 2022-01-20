---
title: 'How to configure Fail2Ban in Ubuntu 20.04 with Threat Jammer'
excerpt: 'Fail2Ban is a security system that helps protect your server from malicious attacks. This article explains how to configure Fail2Ban with Threat Jammer in Ubuntu 20.04.'
coverImage: '/tutorialsimg/fail2ban-threatjammer-logo.png'
created: '2021-01-20'
updated: '2021-01-20'
readTime: 2
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-configure-fail2ban-in-ubuntu.md
  home: /tutorials/index
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## What is Fail2Ban?

[Fail2Ban](https://www.fail2ban.org) is an intrusion prevention software framework that protects computer servers from brute-force attacks. Fail2Ban operates by monitoring log files (e.g. `/var/log/auth.log`, `/var/log/apache/access.log`, etc.) for selected entries and running scripts based on them.[5] Most commonly this is used to block selected IP addresses that may belong to hosts that are trying to breach the system's security. It can ban any host IP address that makes too many login attempts or performs any other unwanted action within a time frame defined by the administrator. 

Fail2Ban is typically set up to unban a blocked host within a certain period, so as to not "lock out" any genuine connections that may have been temporarily misconfigured. Fail2Ban can be extended to run custom `actions` when a ban or a unban is triggered. This custom actions can contact a third party software to notify of the blocked IP address. 

This tutorial will show you how to configure a Fail2Ban action to notify Threat Jammer of the blocked IP address.

## Install Fail2Ban

### Ubuntu

The default Ubuntu 20.04 (also previous versions) repositories includes the Fail2Ban package. To install, enter the following command as `root` or user with `sudo` privileges:

```bash 
sudo apt update
sudo apt install fail2ban
```

Once the installation is over, the Fail2ban service will start automatically. You can verify it by checking the status of the service:

```bash
sudo systemctl status fail2ban
```

The output will look like this:

```bash
sudo systemctl status fail2ban
* fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-01-20 11:31:26 UTC; 1h 40min ago
       Docs: man:fail2ban(1)
    Process: 10025 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 10027 (f2b/server)
      Tasks: 5 (limit: 2280)
     Memory: 13.1M
     CGroup: /system.slice/fail2ban.service
             └─10027 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Jan 20 11:31:26 fail2ban-server systemd[1]: Starting Fail2Ban Service...
Jan 20 11:31:26 fail2ban-server systemd[1]: Started Fail2Ban Service.
Jan 20 11:31:26 fail2ban-server fail2ban-server[10027]: Server ready
```

The Fail2Ban service is running. It's time to start configuring it to send notifications to Threat Jammer.

## Configure the ThreatJammer action

The Fail2Ban action is file you can find in this [gist file](https://gist.github.com/diegoparrilla/d1869ef58ce3bf551ddfd977283e0c9d). The file must be placed in the `/etc/fail2ban/action.d` directory. You can run this command from the directory where the file should be located to download it:

```bash
curl https://gist.githubusercontent.com/diegoparrilla/d1869ef58ce3bf551ddfd977283e0c9d/raw/fea2a75160ecc6d270f861bbe04371803c019afb/threatjammer.com --output threatjammer.conf
``` 

Once we have the action in the right folder, we need to modify the file `/etc/fail2ban/jail.local` to include the new action. If the file doesn't exist, you can create it by copying the file `/etc/fail2ban/jail.conf` to `/etc/fail2ban/jail.local`.

Now find these lines in the file `/etc/fail2ban/jail.local`:

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_)s
```

and add the `action_threatjammer` line below as follows:

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_)s
	 %(action_threatjammer)s[threatjammer_apikey="YOUR_API_KEY"]

```

You can obtain an API key signing up from the [Threat Jammer website](https://threatjammer.com/). Replace `YOUR_API_KEY` with your API key.

## Enable the ThreatJammer action

To enable the action, you need to restart the Fail2Ban service:

```
sudo systemctl restart fail2ban
```

You can also simply reload the Fail2Ban service with the client tool:

```
sudo fail2ban-client reload
```

## Testing the configuration

If the configuration is correct, you can test it by running the following command:

```
$ sudo fail2ban-client get sshd actions
```

it should output something like this:

```
The jail sshd has the following actions:
iptables-multiport, threatjammer
```

The list of actions should contain the `threatjammer` action.

To watch the `ban` command, type the following command:

```
fail2ban-client get sshd action threatjammer actionban
```

it should show the full command. The API KEY has been redacted.

```
curl -X 'POST' http://paris.report.threatjammer.com/v1/ip -H 'accept: application/json' -H 'Authorization: Bearer YOUR_API_KEY' -H 'Content-Type: application/json' -d '{ "addresses": [ "<ip>" ], "ttl": 86400, "type": "ABUSE", "tags": ["FAIL2BAN"] }'
```

To display the `unban` command, type:

```
fail2ban-client get sshd action threatjammer actionunban
```

## Using the ThreatJammer User API to view banned IPs

The User API can be used to view banned IPs, update the TTL, and remove the ban. To use it, you need to sign up for an account at [Threat Jammer](https://threatjammer.com/) for free. Once you have an account, you will obtain an API Key. See how to manage the API Key in the [Threat Jammer documentation](/docs/threat-jammer-api-keys).

The different endpoints to manage the reported IP addresses are in the [Denylist data query and management](https://paris.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management) section of the API documentation.

To view all the banned (or reported, using the Report API nomenclature) IP addresses, you can use the following command:

```bash
curl -X 'GET' \
  'https://REGION.api.threatjammer.com/v1/denylist/reported/ip' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

the ouput is a JSON object with the following structure:

```JSON
{
  "self": "/v1/denylist/reported/ip",
  "addresses": [
    {
      "self": "/v1/denylist/reported/ip/193.122.61.187",
      "last_report": 1642678418000,
      "total_reports": 2,
      "dataset": "/v1/dataset/ip/ABUSE",
      "tags": [
        "FAIL2BAN"
      ],
      "expiry": 1642764818000,
      "protocol": "IPV4"
    },
    {
      "self": "/v1/denylist/reported/ip/82.65.23.62",
      "last_report": 1642689906000,
      "total_reports": 1,
      "dataset": "/v1/dataset/ip/ABUSE",
      "tags": [
        "FAIL2BAN"
      ],
      "expiry": 1642776306000,
      "protocol": "IPV4"
    }
  ]
}
```

You can learn about the JSON object fields in the [endpoint definition](https://paris.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/query_all_the_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_get). This endpoint has different filtering capabilities and can export the information in JSON, CSV and [AWS-WAF format](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/wafv2/get-ip-set.html).

If you want delete all the reported IP addresses, you can use the following command as described in the [Delete all](https://paris.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/delete_all_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_all_delete) endpoint of the API documentation:

```bash
curl -X 'DELETE' \
  'https://REGION.api.threatjammer.com/v1/denylist/reported/ip/all' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

*If you need help you can try first in our [community site](/community), or our [support services](/support)*
