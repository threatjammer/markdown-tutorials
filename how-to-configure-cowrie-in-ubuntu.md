---
title: 'How to configure Cowrie in Ubuntu 20.04 with Threat Jammer'
excerpt: 'Cowrie is a honeypot designed to log brute force attacks and the shell interaction performed by the attacker. This article explains how to configure Cowrie with Threat Jammer in Ubuntu 20.04.'
coverImage: '/tutorialsimg/cowrie-threatjammer-logo.png'
created: '2021-01-21'
updated: '2021-01-21'
readTime: 2
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-configure-cowrie-in-ubuntu.md
  home: /tutorials/index
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## What is Cowrie?

Cowrie is a medium to high interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker

## Install Fail2Ban

### Ubuntu


## Configure the ThreatJammer action


You can obtain an API key signing up from the [Threat Jammer website](https://threatjammer.com/). Replace `YOUR_API_KEY` with your API key.

## Enable the ThreatJammer action


## Testing the configuration


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
