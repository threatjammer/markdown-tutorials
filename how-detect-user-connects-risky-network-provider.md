---
title: 'How to detect if a user connects from a risky network provider'
excerpt: 'Some network providers are lousy at dealing with malicious actors on their network or data centers, but with Threat Jammer you can find out how prone an Autonomous System is to host them.'
coverImage: '/tutorialsimg/detect-risky-network-provider.png'
created: '2022-04-11'
updated: '2022-06-08'
readTime: 2
version: '1.0.0'
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-detect-user-connects-risky-network-provider.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## What is a risky network provider?

Every day there are countless attempts to compromise the security of a server or a personal device such as a cell phone or PC. And a significant percentage of those attempts succeed, becoming a malicious element ready to be used as a spearhead for evil actions. Suppose the network provider implements adequate preventive measures. In that case, it can detect such an intrusion (e.g., by detecting abnormal traffic) and either fix the problem by itself or ask the device owner to fix it. That is what we all expect from a provider with good practices in place.

But sometimes, this is not what happens, and the network service provider does not have these best practices in place for various reasons. Or even worse, it is an unscrupulous provider hosting malicious actors regardless of the damage it can do to third parties. These are the providers for whom Threat Jammer implements a risk calculation system.

> **Therefore, a connection from a network service with a Threat Jammer high-risk score can qualify as suspicious activity.**

## What is the Threat Jammer risk score system?

**In Threat Jammer, every resource relevant to the threat detection has a risk score along with it**. The risk score is a number between 0 and 100, where 0 is the lowest value meaning the resource is not a threat, and 100 is the highest value meaning the resource is a threat. To help humans to understand the risk score, we have created a human-readable risk score scale explained below:

| Risk Score | Description |
| ---------- | ----------- |
| 0 - 34     | Low risk    |
| 35 - 68    | Medium risk |
| 69 - 100   | High risk   |

If you are familiar with our website, each risk score also has a color associated with it:

| Risk Score | Color |
| ---------- | ----- |
| 0 - 34     | Green |
| 35 - 68    | Yellow |
| 69 - 100   | Red |

For the Autonomous Systems and Datacenters, Threat Jammer calculates the score as follows:
- Overall risk score for the Autonomous System or Datacenter, and
- Overall risk score for each of the prefix subnets of the Autonomous Systems or Datacenters.

The risk score for each of the prefix subnets is a density function that take as parameters the number of IP addresses of malicious actors in the subnet and the individual risk score of each malicious actor. So, the more malicious actors in the subnet, the higher the risk score for the subnet.

The risk score for the Autonomous System or Datacenter is the average of the risk scores for the prefix subnets considering the subnets' size.

## User API endpoints

> **You need an API Key to access the endpoints. Sign up for free at [Threat Jammer](https://threatjammer.com) and get your API Key.**

Thanks to the [User API](https://dublin.api.threatjammer.com/docs), there are several ways to detect if a user comes from a risky Autonomous System or Datacenter with Threat Jammer:

### Option 1: Using the assessment endpoint

The assessment endpoint returns a **risk or confidence score for the user's IP address**. The score follows a [multi-factor approach](https://threatjammer.com/docs/how-threat-jammer-works). Several factors are if the IP address belongs to a risky Autonomous System or Datacenter or the prefix subnets of an Autonomous Systems or Datacenters. The score will be the highest value of the four (AS risk, Datacenter risk, AS prefix risk, Datacenter prefix risk).

You can use the following `curl` from the command line to get the risk score of a user's IP address. In this example, we will use an IP address of an Autonomous System with a high risk score:

```
curl -X 'GET' \
  'https://dublin.api.threatjammer.com/v1/asses/ip/2.58.149.174' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY'
```

Response:

```
{
  "self": "/v1/assess/ip/2.58.149.174",
  "score": 90,
  "risk": "HIGH",
  "datasets": [],
  "sources": [],
  "first_appearance": [],
  "last_appearance": [],
  "asn": "/v1/asn/399471",
  "asn_prefix": "/v1/asn/prefix/2.58.148.0%252F22",
  "reason": "Found in an ASN prefix with risky score.",
  "denylisted": "",
  "allowlisted": "",
  "datacenter": "",
  "datacenter_prefix": ""
}
```

The `score` and `risk` fields of the JSON response contain the risk of the IP address. Threat Jammer did not find the IP address was not in any denylist or any user reported as a malicious IP address, but the risk score is high. The `reason` field contains the reason for the risk score.

> **It's also possible to [search for the IP address in the Threat Jammer site](https://threatjammer.com/info/2.58.149.174) or use the [Testing site of the User API](https://dublin.api.threatjammer.com/docs#/Data%20assesment/assess_ip_v1_assess_ip__ip_address__get).**

### Option 2: Using the Autonomous System and Datacenter endpoints

The Autonomous System and Datacenter endpoints return detailed information about the Autonomous Systems, Datacenters, and network prefixes that host the IP address. These endpoints return a complete list of details, so this endpoint is helpful for researchers, analysts, forensics, and other people who want to learn more about the context where IP addresses come.

To [get the details of the Autonomous System prefix of the previous user's IP address](https://dublin.api.threatjammer.com/docs#/Autonomous%20Systems%20information/query_asn_prefix_information_v1_asn_prefix_post):

```
curl -X 'POST' \
  'https://dublin.api.threatjammer.com/v1/asn/prefix' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"prefix": "2.58.149.174"}'
```

Response:

```
{
  "self": "/v1/asn/prefix/2.58.148.0%252F22",
  "asn": "/v1/asn/399471",
  "object_type": "ipv4",
  "mantainer": "Serverion LLC",
  "description": "",
  "registry_date": "20190321",
  "registry_status": "/v1/asn/status/allocated",
  "score": 90,
  "risk": "HIGH"
}
```

The JSON object contains information about the prefix and the Autonomous System it belongs. A developer can use the information obtained in the `asn` to get the details of the Autonomous System.

A request to the endpoint with an IP address not in any Autonomous System will [return a 404 error and a JSON object](https://threatjammer.com/docs/error-resource-not-found) with the following fields:

```
{
  "title": "Resource not found",
  "detail": "'IP_ADDRESS' not found. Check the value for a typo or incorrect value.",
  "type:": "https://threatjammer.com/docs/error-resource-not-found"
}
```

It's highly unlikely to find an IP address that is not in any Autonomous System.


## What's next?

Obtaining the risk score of the network provider is a great way to get an idea of the IP address' risk. Even if the risk score assessment result is not entirely positive, it's still a good idea to get a glimpse of the risk. 

If you want to keep learning, [read our documentation](https://threatjammer.com/docs/index) and mainly [check out the possibilities of our API](https://dublin.api.threatjammer.com/docs).


*If you need help, you can try first in our [community site](/community) or our  support services.*

