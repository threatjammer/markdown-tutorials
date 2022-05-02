---
title: 'How to configure Cowrie Honeypot with Threat Jammer'
excerpt: 'Cowrie is a honeypot designed to log brute force attacks and the shell interaction performed by the attacker. This article explains how to configure Cowrie with Threat Jammer.'
coverImage: '/tutorialsimg/cowrie-threatjammer-logo.png'
created: '2022-05-02'
updated: '2022-05-02'
readTime: 2
version: '1.0.0'
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-configure-cowrie-honeypot.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## What is Cowrie?

[Cowrie](https://cowrie.readthedocs.io/en/latest/README.html) is a medium to high interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker. In medium interaction mode (shell) it emulates a UNIX system in Python, in high interaction mode (proxy) it functions as an SSH and telnet proxy to observe attacker behavior to another system.

## What is a honeypot?

A honeypot is a system designed to be placed on a network for the purpose of having attackers to connect to it. Connections to the system are usually not legitimate and allow a network defender to detect the attacker through detailed logging. The logging can reveal not only normal connection information but also session information revealing the techniques, tactics and procedures (TTP) used by the attacker.

## Why connect Threat Jammer to a Cowrie honeypot?

The Threat Jammer output module feeds the private denylists with successful and unsuccessful login attempts thanks to the asynchronous [Reporting API](https://threatjammer.com/docs/introduction-threat-jammer-report-api) of Threat Jammer. The user can use the denylists in their own infrastructure firewalls or WAFs, or use it for forensic analysis.

When users feed Threat Jammer with their data using the Reporting API, they increase the chance of finding malicious activity and reducing the false positives. Crowdsourced Intelligence means that our users can report their data to the Threat Jammer system, and Threat Jammer will assess the data and decide whether aggregate the data to the crowdsourced intelligence database. If the data has enough quality and is ready to consume, Threat Jammer will share it with the community.

## How to configure the Threat Jammer output module 

### Prerequisites - Get an API Key

To use Threat Jammer users need an API KEY obtained after signing up. If you have not signed up yet, please follow the instructions [here](https://threatjammer.com/docs/threat-jammer-api-keys). The access to the API Key is free forever.

### Prerequisites - Install a Cowrie honeypot

The Threat Jammer output module is now part of the Cowrie main branch of the project. 

To use it, you need to install or upgrade to the *latest version* of the Cowrie honeypot. You can learn how to install Cowrie in the [official documentation of the project](https://cowrie.readthedocs.io/en/latest/INSTALL.html).

A good alternative to the standard installation is to use the [Cowrie Docker image](https://hub.docker.com/r/cowrie/cowrie/). 

### Configure the Threat Jammer output module

Once the developers have obtained an API Key, they have to open the configuration file `etc/cowrie.cfg` of Cowrie:

```
[output_threatjammer]
enabled = false
bearer_token = THREATJAMMER_API_TOKEN
#api_url=https://dublin.report.threatjammer.com/v1/ip
#track_login = true
#track_session = false
#ttl = 86400
#category = ABUSE
#tags = COWRIE,LOGIN,SESSION
and edit the section [output_threatjammer] as follows:
```

And change the `enabled` parameter to `true`.

Change the `THREATJAMMER_API_TOKEN` placeholder with the real API Key of the user in the `bearer_token` parameter.

[Re]start Cowrie. If the output module was initialized succeessfully, the user will see the following message in the Cowrie log:

```
[-] ThreatJammer.com output plugin successfully initialized. Category=ABUSE. TTL=86400. Session Tracking=False. Login Tracking=True
```


### Advanced configuration

If you want to configure the output module to send the data to a different Threat Jammer region or change some of the parameters given as default, you can consult the official documentation of the [Threat Jammer output module](https://cowrie.readthedocs.io/en/latest/threatjammer/README.html).


## Testing the configuration

Cowrie logs all the activity of the honeypot and it also logs the activity of the Threat Jammer output module. The output module will send the information every 60 seconds or if the buffer of pending IP addresses to send to the Report API is 1000. The first condition will trigger the send action to the Report API server with a log like this:

```
[HTTP11ClientProtocol (TLSMemoryBIOProtocol),client] ThreatJammer.com output plugin successfully reported {'addresses': ['X.X.X.X', 'Y.Y.Y.Y', 'Z.Z.Z.Z'], 'type': 'ABUSE', 'ttl': 86400, 'tags': ['COWRIE', 'LOGIN', 'SESSION']}.
```

## Using the ThreatJammer User API to view banned IPs

The User API can be used to view banned IPs, update the TTL, and remove the ban. To use it, you need to sign up for an account at [Threat Jammer](https://threatjammer.com/) for free. Once you have an account, you will obtain an API Key. See how to manage the API Key in the [Threat Jammer documentation](/docs/threat-jammer-api-keys).

The different endpoints to manage the reported IP addresses are in the [Denylist data query and management](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management) section of the API documentation.

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

You can learn about the JSON object fields in the [endpoint definition](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/query_all_the_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_get). This endpoint has different filtering capabilities and can export the information in JSON, CSV and [AWS-WAF format](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/wafv2/get-ip-set.html).

If you want delete all the reported IP addresses, you can use the following command as described in the [Delete all](https://dublin.api.threatjammer.com/docs#/Denylist%20data%20query%20and%20management/delete_all_ip_addresses_reported_by_the_user_v1_denylist_reported_ip_all_delete) endpoint of the API documentation:

```bash
curl -X 'DELETE' \
  'https://REGION.api.threatjammer.com/v1/denylist/reported/ip/all' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

*If you need help you can try first in our [community site](/community), or our [support services](/support)*
