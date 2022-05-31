---
title: 'How to send SSH login notifications to Slack with the risk score'
excerpt: 'This article explains how to set up your servers to send the risk score to a Slack channel with an SSH login.'
coverImage: '/tutorialsimg/ssh-threatjammer-logo.png'
created: '2022-06-01'
updated: '2022-06-01'
readTime: 2
version: '1.0.0'
navigation:
  github: https://github.com/threatjammer/markdown-tutorials/blob/main/how-to-use-twitter-information-bot.md
  home: /tutorials/main
  previous: 
  next: 
authors:
  - name: Diego Parrilla
    link: 'https://github.com/orgs/threatjammer/people/diegoparrilla'
ogImage:
  url: ''
---

## Why SSH login notifications?

SSH is probably the most pervasive tool of any *NIX System Administrator to access their managed servers. As a rule of thumb, any Sysadmin should enforce several security policies to prevent SSH brute force attacks. The most obvious is to disable SSH login via password, only allowing login via public/private keys. An excellent practice is to set up notifications to report to the Sysadmins when users log in to the servers.

This short tutorial will show you how to set up a Slack channel to receive immediate notifications of SSH logins with information about the remote connection enriched with the risk score calculated by Threat Jammer. If someone logs in to your server, you will have the chance to take a fast response to the intrusion and to take action.


## Pre-requisites

In this tutorial we will show how to modify any kind of Linux server to receive Slack notifications of SSH logins with the Threat Jammer risk score. Before we start we need to prepare the server and Slack.

### Create a Threat Jammer account and obtain a API Key

This tutorial will show how to modify any Linux server to receive Slack notifications of SSH logins with the Threat Jammer risk score. Before we start, we need to prepare the server and Slack.

### Create a Threat Jammer account and obtain an API Key

The current version of Threat Jammer supports only one API key per user in the only existing region in [Dublin](https://dublin.api.threatjammer.com). When a user register in Threat Jammer, he will automatically receive an *API Key*. The Threat Jammer site has a page with all the information about the *API Keys*:

> [`https://threatjammer.com/keys`](https://threatjammer.com/keys)

This page is only available for users who have registered in Threat Jammer. **So, if you are not a user yet, you can create an API key on the [Threat Jammer website](https://threatjammer.com/api/signup).** The signup is free, and we don't take any personal information or the payment method. 

![Threat Jammer API Key menu](/docsimg/api-keys-menu.png)

You will need the API Key in the following steps, so keep it at hand.

### Install the required packages in the Linux server(s)

To extract information from the JSON output of the Threat Jammer API, we will need to install the package ```jq```.

To install the package in Debian-based Linux systems, you can use the following command:

```
sudo apt-get install jq
```

And in RedHat-based Linux systems, you can use the following command:

```
sudo yum install jq
```

To query the Threat Jammer API, you must install the package ```curl```.

To install the package in Debian-based Linux systems, you can use the following command:

```
sudo apt-get install curl
```

And following the same pattern in RedHat-based Linux systems, use the following command:

```
sudo yum install curl
```

### Create a new Slack App to receive notifications as incoming webhooks

You will need to create a new Slack App to receive notifications as incoming webhooks. The Slack App will receive notifications in the channel specified. To create a new Slack App and enable the incoming webhooks, follow the instructions in the [Incoming webhooks creation guide](https://slack.com/help/articles/115005265063-Incoming-webhooks-for-Slack).

You will probably have to create a new Slack App in your workspace to receive notifications as incoming webhooks. After creating the Slack App, you will need to go to *Features > Incoming webhooks* and enable the *Incoming webhooks* option. Below, in the *Webhooks URLs for Your Workspace* section, click on the *Add New Webhook to Workspace* button and select the channel you want to receive notifications. 

You will need the *Webhook URL* and the *Channel* to configure the shell script. So keep them at hand.

## Configure the Linux server

You will have to copy the following code snippet at the end of your ```~/.bashrc``` file in the remote server:


```
IP="$(echo $SSH_CONNECTION | cut -d " " -f 1)"
if [ ! -z "$IP" ]; then
  TJ_API_KEY="THREAT_JAMMER_API_KEY"
  TJ_URL_PREFIX="https://dublin.api.threatjammer.com"
  SLACK_WEBHOOK_URL="SLACK_APP_INCOMING_WEBHOOK"
  SLACK_CHANNEL="SLACK_CHANNEL_OF_THE_APP_WITH_PUBLISHING_PERMISSIONS"
  SLACK_RISK_COLOR="#02ff00" # Green bar

  HOSTNAME=$(hostname -f)
  NOW=$(date +"%r (UTC %Z), %e %b %Y")

  ASSESS="$(curl -s -X GET ''$TJ_URL_PREFIX'/v1/assess/ip/'$IP'' -H 'accept: application/json' -H 'Authorization: Bearer '$TJ_API_KEY'')"
  RESULT=$(echo $ASSESS)

  SCORE="$(echo $RESULT | jq -r .score)"
  RISK="$(echo $RESULT | jq -r .risk)"
  REASON="$(echo $RESULT | jq -r .reason)"
  RISK_INFO="$SCORE ($RISK) - $REASON"
  ASN="$(echo $RESULT | jq -r .asn)"

  if [ $SCORE -gt 34 ]; then
    SLACK_RISK_COLOR="#ffe200" # Yellow bar
  fi

  if [ $SCORE -gt 67 ]; then
    SLACK_RISK_COLOR="#ff0002" # Red bar
  fi

  AS="$(curl -s -X GET ''$TJ_URL_PREFIX$ASN'' -H 'accept: application/json' -H 'Authorization: Bearer '$TJ_API_KEY'')"
  RESULT=$(echo $AS)
  AS_INFO="AS${ASN:8} $(echo $RESULT | jq -r .name) - $(echo $RESULT | jq -r .description)"

  GEO="$(curl -s -X GET ''$TJ_URL_PREFIX'/v1/geo/'$IP'' -H 'accept: application/json' -H 'Authorization: Bearer '$TJ_API_KEY'')"
  RESULT=$(echo $GEO)
  COUNTRY_CODE="$(echo $RESULT | jq -r .country_iso_code)"
  REGION_NAME="$(echo $RESULT | jq -r .region_name)"
  CITY_NAME="$(echo $RESULT | jq -r .city_name)"
  GEO_INFO="$CITY_NAME - $REGION_NAME ($COUNTRY_CODE)"

  curl -H 'Content-type: application/json' --data '{"attachments":[{"title":"New SSH login of >'"$(whoami)"'< to >'"$HOSTNAME"'< at '"$NOW"'", "color":"'"$SLACK_RISK_COLOR"'", "mrkdwn_in": ["text"], "text": "*Connected from IP:* '"$IP"'\n*Connected from ISP:* '"$AS_INFO"'\n*IP Location:* '"$GEO_INFO"'\n*Threat Jammer Risk:* '"$RISK_INFO"'"}]}' $SLACK_WEBHOOK_URL
fi
```

You can find the code in the following [Gist file](https://gist.github.com/diegoparrilla/d9d548a7729a1a4188def85479833251).

Now you should modify the environment variables ```TJ_API_KEY```, ```SLACK_WEBHOOK_URL``` and ```SLACK_CHANNEL``` with the values obtained from the Threat Jammer website and the Slack App.

Save the file, log out and try to login again. You should see the following message in the Slack channel configured:

![SSH login notification slack no risk](/tutorialsimg/slack-ssh-login-norisk.png)

The green bar means that the risk of the IP is low. The yellow bar means that the risk of the IP is medium, and the red bar means that the risk of the IP is high:

![SSH login notification slack risk high](/tutorialsimg/slack-ssh-login-highrisk.png)

## What's next?

This script is only an example of how simple integration of Threat Jammer API with any service is. Feel free to create a new service and share it with the community. More services could be helpful for you to integrate our API. Why don't you give it a try?

If you want to keep learning, [read our documentation](https://threatjammer.com/docs/index) and mainly [check out the possibilities of our API](https://dublin.api.threatjammer.com/docs).

*If you need help you can try first in our [community site](/community), or our [support services](/support)*

<a href="https://www.flaticon.es/iconos-gratis/ssh" title="ssh iconos">Ssh iconos creados por Freepik - Flaticon</a>
