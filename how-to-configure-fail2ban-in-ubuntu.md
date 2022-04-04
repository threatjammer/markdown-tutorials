---
title: 'How to configure Fail2Ban in Ubuntu 20.04 with Threat Jammer'
excerpt: 'Fail2Ban is a security system that helps protect your server from malicious attacks. This article explains how to configure Fail2Ban with Threat Jammer in Ubuntu 20.04.'
coverImage: '/tutorialsimg/fail2ban-threatjammer-logo.png'
created: '2021-04-04'
updated: '2021-04-04'
readTime: 2
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-configure-fail2ban-in-ubuntu.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## What is Fail2Ban?

[Fail2Ban](https://www.fail2ban.org) is an intrusion prevention software framework that protects computer servers from brute-force attacks. Fail2Ban operates by monitoring log files (e.g. `/var/log/auth.log`, `/var/log/apache/access.log`, etc.) for selected entries and running scripts based on them.[5] Most commonly, it blocks selected IP addresses that may belong to hosts trying to breach the system's security. It can ban any host IP address that makes too many login attempts or performs any other unwanted action within a time frame defined by the administrator. 

A Fail2Ban typical setup also unbans a blocked host within a certain period to not "lockout" any genuine connections banned by mistake or too rigorous rules. Fail2Ban extensions can run custom `actions` when triggered by a ban or unban. These custom actions can contact a third-party software to notify the blocked IP address. 

This tutorial will show you how to configure a Fail2Ban action to notify Threat Jammer of the blocked IP address.

## Why Fail2Ban and Threat Jammer?

Threat Jammer can keep track of the IP addresses banned and unbanned by Fail2Ban in an automated and centralized way. Keeping in a single place all the IP addresses reported by multiple Fail2Ban servers makes it easier to analyze and manage the security of your infrastructure. When a new IP address is banned, Threat Jammer will automatically perform the following actions:

- Store the IP address in a deny list private to the user reporting it.
- The User API will return the maximum risk score for the banned IP addresses.
- Keep track of all the IP addresses banned. This dataset can be downloaded in different formats at any time, for example, to import into a Firewall, Router, or Web Application Firewall.
- Crowd security: if more users report the same IP addresses in a specific period, Threat Jammer will automatically ban the IP address for all the platform users.

## Install Fail2Ban

### Ubuntu

The default Ubuntu 20.04 (also previous versions) repositories include the Fail2Ban package. To install, enter the following command as `root` or user with `sudo` privileges:

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

You can find the Fail2Ban action file in this [gist file](https://gist.github.com/diegoparrilla/d1869ef58ce3bf551ddfd977283e0c9d). You must place this file in the `/etc/fail2ban/action.d` directory. You can run this command from this directory to download it:

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

You can obtain an API key by signing up from the [Threat Jammer website](https://threatjammer.com/). Replace `YOUR_API_KEY` with your API key.

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

It should show the whole command (we redacted the API KEY).

```
curl -X 'POST' https://dublin.report.threatjammer.com/v1/ip -H 'accept: application/json' -H 'Authorization: Bearer YOUR_API_KEY' -H 'Content-Type: application/json' -d '{ "addresses": [ "<ip>" ], "ttl": 86400, "type": "ABUSE", "tags": ["FAIL2BAN"] }'
```

To display the `unban` command, type:

```
fail2ban-client get sshd action threatjammer actionunban
```

## Using the ThreatJammer User API to view banned IPs

Developers can use the User API to view banned IPs, update the TTL, and remove the ban. To use it, you need to sign up for an account at [Threat Jammer](https://threatjammer.com/) for free. Once you have an account, you will obtain an API Key. See how to manage the API Key in the [Threat Jammer documentation](/docs/threat-jammer-api-keys).

The different endpoints to manage the reported IP addresses are in the [Denylist data query and management](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management) section of the API documentation.

To view all the banned (or reported, using the Report API nomenclature) IP addresses, you can use the following command:

```bash
curl -X 'GET' \
  'https://REGION.api.threatjammer.com/v1/denylist/reported/ip' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

the output is a JSON object with the following structure:

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

You can learn about the JSON object fields in the [endpoint definition](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/query_all_the_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_get). This endpoint has different filtering capabilities and can export the information in JSON, CSV and [AWS-WAF format](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/wafv2/get-ip-set.html).

If you want to delete all the reported IP addresses, you can use the following command as described in the [Delete all](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/delete_all_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_all_delete) endpoint of the API documentation:

```bash
curl -X 'DELETE' \
  'https://REGION.api.threatjammer.com/v1/denylist/reported/ip/all' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

## What's next?

The Report API lets users automate multiple reporting tasks from their platforms to Threat Jammer. If you want to keep learning, [read our documentation](https://threatjammer.com/docs/index) and mainly [check out the possibilities of the Report API](https://dublin.report.threatjammer.com/docs).

*If you need help, you can try first in our [community site](/community) or our [support services](/support)*
