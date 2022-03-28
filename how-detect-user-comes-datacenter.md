---
title: 'How to detect if a user comes from a Datacenter'
excerpt: 'A good way to detect if an actor can be malicious is to check if the user´s IP address comes from a Datacenter with Threat Jammer.'
coverImage: '/tutorialsimg/detect-datacenter-ip-address.png'
created: '2022-03-28'
updated: '2022-03-28'
readTime: 2
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-detect-user-comes-datacenter.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## A Datacenter IP address is a powerful signal for abuse detection

When a user connects to an online service, they do it from a device such as a cell phone, tablet, PC, or laptop. These devices connect to the web through internet service providers intended for residential, mobile, or enterprise audiences. So we can infer that most of the connections made by humans will come from these IP addresses.

When a server responds to a request from users' devices or connects to another server (machine-to-machine communication), it does so over different IP address ranges because the datacenters have pools optimized to connect servers to the service providers' backbone. 

> **Therefore, a connection from a device generally used by humans made from a data center can qualify as suspicious activity.**

## Detecting if the IP address comes from a Datacenter

Detecting whether an IP address belongs to a data center, there are usually three options:

1. Use an OSINT dataset with the IP address ranges of datacenters. A good example was [IPCAT](https://github.com/client9/ipcat/), but sadly it is not updated often (the last time was in 2019).
2. Use lists of residential and mobile IP address ranges. If the IP address of your user is in one of these ranges, likely, the user is not from a data center. This is not a perfect solution, and it is not the best way to detect if the user comes from a data center because the chances of false positives are higher.
3. Use a CSINT dataset with the IP address ranges of datacenters. It is the best solution because the source updates daily, and it is the most accurate way to avoid false positives.

**[Threat Jammer](https://threatjammer.com) follows the CSINT approach, building our dataset from different sources.**

## User API endpoints

> **You need an API Key to access the endpoints. Sign up for free at [Threat Jammer](https://threatjammer.com) and get your API Key.**

Thanks to the [User API](https://dublin.api.threatjammer.com/docs), there are two ways to detect if a user comes from a Datacenter with Threat Jammer:

### Option 1: Using the assessment endpoint

The assessment endpoint returns a **risk or confidence score for the user's IP address**. The score follows a [multi-factor approach](https://threatjammer.com/docs/how-threat-jammer-works), and one of the factors is if the IP address is in a data center. 

You can use the following `curl` from the command line to get the risk score of a user's IP address. In this example, we will use a random IP address of Microsoft Azure:

```
curl -X 'GET' \
  'https://dublin.api.threatjammer.com/v1/asses/ip/20.36.32.1' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

Response:

```
{
  "self": "/v1/assess/ip/20.36.32.1",
  "score": 1,
  "risk": "LOW",
  "datasets": [],
  "sources": [],
  "first_appearance": [],
  "last_appearance": [],
  "asn": "/v1/asn/8075",
  "asn_prefix": "/v1/asn/prefix/20.36.0.0%252F15",
  "reason": "Found in an Datacenter network with risky score.",
  "denylisted": "",
  "allowlisted": "",
  "datacenter": "/v1/datacenter/MICROSOFT_CORPORATION",
  "datacenter_prefix": "/v1/datacenter/prefix/20.0.0.0%252F10",
}
```

The `datacenter` field of the JSON response contains the URI of the datacenter that hosts the IP address. This field will be empty if the IP address is not in a data center.

> **It's also possible to [search for the IP address in the Threat Jammer site](https://threatjammer.com/info/20.36.32.1) or use the [Testing site of the User API](https://dublin.api.threatjammer.com/docs#/Data%20assesment/assess_ip_v1_assess_ip__ip_address__get).**


### Option 2: Using the datacenter endpoint

The data center endpoint returns detailed information about the data center that hosts the IP address. Strictly speaking, you don't need to know many details about the data center, so this endpoint is helpful for researchers, analysts, forensics, and other people who want to learn more about the IP address.

To [get the details of the datacenter of the previous user's IP address](https://dublin.api.threatjammer.com/docs#/Datacenter%20information/query_IP_address_network_information_v1_datacenter_ip__ip_address__get):

```
curl -X 'GET' \
  'https://dublin.api.threatjammer.com/v1/datacenter/ip/20.36.32.1' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

Response:

```
{
  "self": "/v1/datacenter/prefix/20.0.0.0%252F10",
  "datacenter": "/v1/datacenter/MICROSOFT_CORPORATION",
  "min_score": 30,
  "max_score": 90,
  "ip_abuse_total": 869,
  "score": 1,
  "risk": "LOW",
}
```

The JSON object contains not only information about the data center but also information about the entire prefix the IP address belongs to and information relevant to the risk score:
- The number of IP addresses with a risk score higher than 0.
- The maximum risk score of the prefix.
- The minimum risk score of the prefix (of the IP addresses with a score higher than 0).
- The average risk score of the prefix.

A request to the endpoint with an IP address not in any datacenter will [return a 404 error and a JSON object](https://threatjammer.com/docs/error-resource-not-found) with the following fields:

```
{
  "title": "Resource not found",
  "detail": "'IP_ADDRESS' not found. Check the value for a typo or incorrect value.",
  "type:": "https://threatjammer.com/docs/error-resource-not-found"
}
```

## What's next?

Figuring out if a user's IP address belongs to a data center and which one can be very relevant to detecting a rogue actor trying to abuse your online platform. If you want to keep learning, [read our documentation](https://threatjammer.com/docs/index) and mainly [check out the possibilities of our API](https://dublin.api.threatjammer.com/docs).


*If you need help, you can try first in our [community site](/community) or our [support services](/support)*
