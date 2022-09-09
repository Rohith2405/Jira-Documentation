# JIRAlert
Build Status Go Report Card GoDoc Slack Prometheus Alertmanager webhook receiver for JIRA.

## OVERVIEW:
JIRAlert implements Alertmanager's webhook HTTP API and connects to one or more JIRA instances to create highly configurable JIRA issues. One issue is created per distinct group key — as defined by the group_by parameter of Alertmanager's route configuration section — but not closed when the alert is resolved. The expectation is that a human will look at the issue, take any necessary action, then close it. If no human interaction is necessary then it should probably not alert in the first place. This behavior however can be modified by setting auto_resolve section, which will resolve the jira issue with required state.

If a corresponding JIRA issue already exists but is resolved, it is reopened. A JIRA transition must exist between the resolved state and the reopened state — as defined by reopen_state — or reopening will fail. Optionally a "won't fix" resolution — defined by wont_fix_resolution — may be defined: a JIRA issue with this resolution will not be reopened by JIRAlert.

# Jira setup

## step-by-step procedure:

*1. Create jira free account
*2. Create ==service desk management== project
*3. Add project name and issue type
*4. Create a new token in jira
*5. Copy the project name and  key for later use


# Requierment:
Create on folder and copy the files in this link
[Github Link](https://github.com/prometheus-community/jiralert)



### CONFIGURATION(Alertmanager):
The configuration file is essentially a list of receivers matching 1-to-1 all Alertmanager receivers using JIRAlert; plus defaults (in the form of a partially defined receiver); and a pointer to the template file.

==Each receiver must have a unique name (matching the Alertmanager receiver name), JIRA API access fields (URL, username and password)==, a handful of required issue fields (such as the JIRA project and issue summary), some optional issue fields (e.g. priority) and a fields map for other (standard or custom) JIRA fields. Most of these may use Go templating to generate the actual field values based on the contents of the Alertmanager notification. The exact same data structures and functions as those defined in the Alertmanager template reference are available in JIRAlert.


# ALERTMANAGER CONFIGURATION:

To enable Alertmanager to talk to JIRAlert you need to configure a webhook in Alertmanager. You can do that by adding a ==webhook receiver to your Alertmanager configuration==.

```
receivers:
- name: 'jira-ab'
  webhook_configs:
  - url: 'http://localhost:9097/alert'
    # JIRAlert ignores resolved alerts, avoid unnecessary noise
    send_resolved: false
```


# Alertmanager.yaml setup

## step-by-step procedure:

*1.Enter tha details(kind,apiversion,metadata,namespace) in alertmanager.yaml for the type of metrics you are using
*2.In receiver add the project you created in jira
*3.Add jira's webhook URL

## sample.yaml

```
apiVersion: operator.victoriametrics.com/v1beta1
kind: VMAlertmanagerConfig
metadata:
  name: jira-config
  namespace: observability
  
spec:
  route:
    #group_wait: 30s
    #group_interval: 5m
    #repeat_interval: 12h
    receiver: 'observability1'
    
     
    
  receivers:
  - name: 'observability1'
    webhook_configs:
    
    - url:  'http://10.105.163.161:9097/alert'
      send_resolved: true
```


# Pod deployment:

## step-by-step procedure:

*1. Open secret.yaml
*2. Enter the details for the following fields
       1.namespace
       2.api_url
       3.user(mailid used in jira)
       4.password(Token)
       5.receiver(alertmanager path name)
       6.project(enter the project key)


# Testing:
JIRAlert expects a JSON object from Alertmanager. The format of this JSON is described in the Alertmanager documentation or, alternatively, in the Alertmanager GoDoc.

To quickly test if JIRAlert is working you can run:

$ curl -H "Content-type: application/json" -X POST \
  -d '{"receiver": "Observability1", "status": "firing", "alerts": [{"status": "firing", "labels": {"alertname": "TestAlert", "key": "value"} }], "groupLabels": {"alertname": "TestAlert"}}' \
  http://localhost:9097/alert



