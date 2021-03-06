# inspectr

this is a binary that, when run in a k8s cluster, gets details of pods in the same cluster (via the k8s master/API), and alerts if certain conditions are met.

currently the only condition that's ever met is "are there any upgrades to the image being run in pod X?"


## result grouping

results are unique by cluster/pod-name/container-name/namespace/image

results are stored (and appear in alerts) in a map where the key is cluster:image:pod-name:cluster-name


## running frequency

there is a daily 'scheduled alert window', within which all results obtained will be outputted

outside of this window, the process runs every minute

if any errors are encountered, the process won't run again for 5 minutes.


## alert cache

to prevent noise, the binary keeps a cache of the clusters/images that an alert has been produced for

if a cluster/image appears in the cache, it won't get alerted on again until the 'scheduled alert window'

when the binary first runs, the cache is empty, so essentially you'll get a full result alert every time a pod starts/restarts


## slack alerts

the binary needs to know the webhook id that you want the alerts going to

it looks for this id in the environment variables:

* ___INSPECTR_SLACK_WEBHOOK_ID___

this id is the string that comes after "https://hooks.slack.com/services/" in your webhook URL

if that's not set, the binary still runs, you just (obviously) won't see any alerts in your Slack channel. Inspectr results are still logged via glog.


## jira

the inspectr binary can create a JIRA detailing the image upgrades it finds, or update existing JIRAs that may have been created on previous runs (it will only update if there are any additional new versions found, though).

to enable this functionality, you'll need to set 2 environment variables:

* ___INSPECTR_JIRA_URL___
  * URL of your JIRA instance
* ___INSPECTR_JIRA_PARAMS___
  * Usage: user|pass|project|issueType|otherFieldKey:otherFieldValue,otherFieldKey:otherFieldValue...
  * mandatory: user, pass, project, issueType
  * oauth2 hasn't been integrated yet..
  * optional (as your JIRA instance may require them): otherFieldKey:otherFieldValue
  * otherFieldKey should equal the field names as they appear in your JIRA UI
  * note the sepratators, "|" and "," and ":"
  
it's recommended to:

* __use kubernetes secrets__ for your environment variables (note if you've got any whitespaces in your otherFieldKeys, you'll have to wrap the environment variable in quotes when you issue the kubectl create command)
* __create a new JIRA user__ for use by inspectr, that has limited access to a single project
* __use https__
* [obvious advice about passwords]
