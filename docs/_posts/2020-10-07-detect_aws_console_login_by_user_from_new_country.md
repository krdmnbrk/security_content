---
title: "Detect AWS Console Login by User from New Country"
excerpt: "Unused/Unsupported Cloud Regions
"
categories:
  - Cloud
last_modified_at: 2020-10-07
toc: true
toc_label: ""
tags:
  - Unused/Unsupported Cloud Regions
  - Defense Evasion
  - Splunk Enterprise
  - Splunk Enterprise Security
  - Splunk Cloud
  - Authentication
---



[Try in Splunk Security Cloud](https://www.splunk.com/en_us/products/cyber-security.html){: .btn .btn--success}

#### Description

This search looks for AWS CloudTrail events wherein a console login event by a user was recorded within the last hour, then compares the event to a lookup file of previously seen users (by ARN values) who have logged into the console. The alert is fired if the user has logged into the console for the first time within the last hour

- **Type**: [Hunting](https://github.com/splunk/security_content/wiki/Detection-Analytic-Types)
- **Product**: Splunk Enterprise, Splunk Enterprise Security, Splunk Cloud
- **Datamodel**: [Authentication](https://docs.splunk.com/Documentation/CIM/latest/User/Authentication)
- **Last Updated**: 2020-10-07
- **Author**: Bhavin Patel, Splunk
- **ID**: 67bd3def-c41c-4bf6-837b-ae196b4257c6


#### Annotations

<details>
  <summary>ATT&CK</summary>

<div markdown="1">


| ID             | Technique        |  Tactic             |
| -------------- | ---------------- |-------------------- |
| [T1535](https://attack.mitre.org/techniques/T1535/) | Unused/Unsupported Cloud Regions | Defense Evasion |

</div>
</details>


<details>
  <summary>Kill Chain Phase</summary>

<div markdown="1">

* Actions on Objectives


</div>
</details>


<details>
  <summary>NIST</summary>

<div markdown="1">

* DE.DP
* DE.AE



</div>
</details>

<details>
  <summary>CIS20</summary>

<div markdown="1">

* CIS 16



</div>
</details>

<details>
  <summary>CVE</summary>

<div markdown="1">


</div>
</details>

#### Search 

```

| tstats earliest(_time) as firstTime latest(_time) as lastTime from datamodel=Authentication where Authentication.signature=ConsoleLogin by Authentication.user Authentication.src 
| iplocation Authentication.src 
| `drop_dm_object_name(Authentication)` 
| table firstTime lastTime user Country 
| join user  type=outer [
| inputlookup previously_seen_users_console_logins 
| stats min(firstTime) AS earliestseen by user Country 
| fields earliestseen user Country] 
| eval userCountry=if(firstTime >= relative_time(now(), "-24h@h"), "New Country","Previously Seen Country") 
| eval userStatus=if(earliestseen >= relative_time(now(),"-24h@h") OR isnull(earliestseen), "New User","Old User") 
| where userCountry = "New Country" AND userStatus != "Old User" 
| `security_content_ctime(firstTime)` 
| `security_content_ctime(lastTime)` 
| table firstTime lastTime user Country  userStatus userCountry  
| `detect_aws_console_login_by_user_from_new_country_filter`
```

#### Macros
The SPL above uses the following Macros:
* [security_content_ctime](https://github.com/splunk/security_content/blob/develop/macros/security_content_ctime.yml)

> :information_source:
> **detect_aws_console_login_by_user_from_new_country_filter** is a empty macro by default. It allows the user to filter out any results (false positives) without editing the SPL.

#### Lookups
The SPL above uses the following Lookups:

* [previously_seen_users_console_logins](https://github.com/splunk/security_content/blob/develop/lookups/previously_seen_users_console_logins.yml) with [data](https://github.com/splunk/security_content/tree/develop/lookups/previously_seen_users_console_logins.csv)

#### Required field
* _time
* Authentication.signature
* Authentication.user
* Authentication.src


#### How To Implement
You must install and configure the Splunk Add-on for AWS (version 5.1.0 or later) and Enterprise Security 6.2, which contains the required updates to the Authentication data model for cloud use cases. Run the `Previously Seen Users in AWS CloudTrail - Initial` support search only once to create a baseline of previously seen IAM users within the last 30 days. Run `Previously Seen Users in AWS CloudTrail - Update` hourly (or more frequently depending on how often you run the detection searches) to refresh the baselines. You can also provide additional filtering for this search by customizing the `detect_aws_console_login_by_user_from_new_country_filter` macro.

#### Known False Positives
When a legitimate new user logins for the first time, this activity will be detected. Check how old the account is and verify that the user activity is legitimate.

#### Associated Analytic story
* [Suspicious AWS Login Activities](/stories/suspicious_aws_login_activities)
* [Suspicious Cloud Authentication Activities](/stories/suspicious_cloud_authentication_activities)




#### RBA

| Risk Score  | Impact      | Confidence   | Message      |
| ----------- | ----------- |--------------|--------------|
| 42.0 | 70 | 60 | User $user$ is logging into the AWS console from Country $Country$ for the first time |


> :information_source:
> The Risk Score is calculated by the following formula: Risk Score = (Impact * Confidence/100). Initial Confidence and Impact is set by the analytic author. 

#### Reference


#### Test Dataset
Replay any dataset to Splunk Enterprise by using our [replay.py](https://github.com/splunk/attack_data#using-replaypy) tool or the [UI](https://github.com/splunk/attack_data#using-ui).
Alternatively you can replay a dataset into a [Splunk Attack Range](https://github.com/splunk/attack_range#replay-dumps-into-attack-range-splunk-server)


* [https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/suspicious_behaviour/abnormally_high_cloud_instances_launched/cloudtrail_behavioural_detections.json](https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/suspicious_behaviour/abnormally_high_cloud_instances_launched/cloudtrail_behavioural_detections.json)



[*source*](https://github.com/splunk/security_content/tree/develop/detections/cloud/detect_aws_console_login_by_user_from_new_country.yml) \| *version*: **1**