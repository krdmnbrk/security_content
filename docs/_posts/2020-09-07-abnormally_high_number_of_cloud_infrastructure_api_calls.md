---
title: "Abnormally High Number Of Cloud Infrastructure API Calls"
excerpt: "Cloud Accounts
, Valid Accounts
"
categories:
  - Cloud
last_modified_at: 2020-09-07
toc: true
toc_label: ""
tags:
  - Cloud Accounts
  - Valid Accounts
  - Defense Evasion
  - Initial Access
  - Persistence
  - Privilege Escalation
  - Defense Evasion
  - Initial Access
  - Persistence
  - Privilege Escalation
  - Splunk Enterprise
  - Splunk Enterprise Security
  - Splunk Cloud
  - Change
---



[Try in Splunk Security Cloud](https://www.splunk.com/en_us/products/cyber-security.html){: .btn .btn--success}

#### Description

This search will detect a spike in the number of API calls made to your cloud infrastructure environment by a user.

- **Type**: [Anomaly](https://github.com/splunk/security_content/wiki/Detection-Analytic-Types)
- **Product**: Splunk Enterprise, Splunk Enterprise Security, Splunk Cloud
- **Datamodel**: [Change](https://docs.splunk.com/Documentation/CIM/latest/User/Change)- **Datasource**: [Splunk Add-on for Amazon Kinesis Firehose](https://splunkbase.splunk.com/app/3719)
- **Last Updated**: 2020-09-07
- **Author**: David Dorsey, Splunk
- **ID**: 0840ddf1-8c89-46ff-b730-c8d6722478c0


#### Annotations

<details>
  <summary>ATT&CK</summary>

<div markdown="1">


| ID             | Technique        |  Tactic             |
| -------------- | ---------------- |-------------------- |
| [T1078.004](https://attack.mitre.org/techniques/T1078/004/) | Cloud Accounts | Defense Evasion, Initial Access, Persistence, Privilege Escalation |

| [T1078](https://attack.mitre.org/techniques/T1078/) | Valid Accounts | Defense Evasion, Initial Access, Persistence, Privilege Escalation |

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
* DE.CM
* PR.AC



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

| tstats count as api_calls values(All_Changes.command) as command from datamodel=Change where All_Changes.user!=unknown All_Changes.status=success by All_Changes.user _time span=1h 
| `drop_dm_object_name("All_Changes")` 
| eval HourOfDay=strftime(_time, "%H") 
| eval HourOfDay=floor(HourOfDay/4)*4 
| eval DayOfWeek=strftime(_time, "%w") 
| eval isWeekend=if(DayOfWeek >= 1 AND DayOfWeek <= 5, 0, 1) 
| join user HourOfDay isWeekend [ summary cloud_excessive_api_calls_v1] 
| where cardinality >=16 
| apply cloud_excessive_api_calls_v1 threshold=0.005 
| rename "IsOutlier(api_calls)" as isOutlier 
| where isOutlier=1 
| eval expected_upper_threshold = mvindex(split(mvindex(BoundaryRanges, -1), ":"), 0) 
| where api_calls > expected_upper_threshold 
| eval distance_from_threshold = api_calls - expected_upper_threshold 
| table _time, user, command, api_calls, expected_upper_threshold, distance_from_threshold 
| `abnormally_high_number_of_cloud_infrastructure_api_calls_filter`
```

#### Macros
The SPL above uses the following Macros:

> :information_source:
> **abnormally_high_number_of_cloud_infrastructure_api_calls_filter** is a empty macro by default. It allows the user to filter out any results (false positives) without editing the SPL.

#### Required field
* _time
* All_Changes.command
* All_Changes.user
* All_Changes.status


#### How To Implement
You must be ingesting your cloud infrastructure logs. You also must run the baseline search `Baseline Of Cloud Infrastructure API Calls Per User` to create the probability density function.

#### Known False Positives


#### Associated Analytic story
* [Suspicious Cloud User Activities](/stories/suspicious_cloud_user_activities)




#### RBA

| Risk Score  | Impact      | Confidence   | Message      |
| ----------- | ----------- |--------------|--------------|
| 15.0 | 30 | 50 | user $user$ has made $api_calls$ api calls, violating the dynamic threshold of $expected_upper_threshold$ with the following command $command$. |


> :information_source:
> The Risk Score is calculated by the following formula: Risk Score = (Impact * Confidence/100). Initial Confidence and Impact is set by the analytic author. 

#### Reference


#### Test Dataset
Replay any dataset to Splunk Enterprise by using our [replay.py](https://github.com/splunk/attack_data#using-replaypy) tool or the [UI](https://github.com/splunk/attack_data#using-ui).
Alternatively you can replay a dataset into a [Splunk Attack Range](https://github.com/splunk/attack_range#replay-dumps-into-attack-range-splunk-server)


* [https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/suspicious_behaviour/abnormally_high_cloud_instances_launched/cloudtrail_behavioural_detections.json](https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/suspicious_behaviour/abnormally_high_cloud_instances_launched/cloudtrail_behavioural_detections.json)



[*source*](https://github.com/splunk/security_content/tree/develop/detections/cloud/abnormally_high_number_of_cloud_infrastructure_api_calls.yml) \| *version*: **1**