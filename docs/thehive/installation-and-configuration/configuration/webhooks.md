# TheHive webhooks

TheHive can notify external system of modification events (case creation, alert update, task assignment, ...). To use webhooks notifications, 2 steps are required: configure a notification, and activate it.

## 1. Define webhook endpoints

The configuration can accept following parameters:

| Parameter                      | Type           | Description                          |
| -------------------------------| -------------- | ------------------------------------ |
| `name`                         | string         | the identifier of the endpoint. It is used when the webhook is setup for an organisation |
| `version`                      | integer        | defines the format of the message. If `version` is `0`, TheHive will send messages with the same format as TheHive3. Currently TheHive only supports version 0. |
| `wsConfig`                     | dict           | the configuration of HTTP client. It contains proxy, SSL and timeout configuration. |
| `includedTheHiveOrganisations` | list of string | list of TheHive organisations which can use this endpoint (default: _all_ (`[*]`) |
| `excludedTheHiveOrganisations` | list of string | list of TheHive organisations which cannot use this endpoint (default: _None_ (`[]`) ) |

The following section should be added in `application.conf` : 

```yaml
## Webhook notification
notification.webhook.endpoints = [
  {
    name: local
    url: "http://127.0.0.1:5000/"
    version: 0
    wsConfig: {}
    includedTheHiveOrganisations: ["*"]
    excludedTheHiveOrganisations: []
  }
]
```

!!! Example

    ```yaml
    ## Webhook notification
    notification.webhook.endpoints = [
      {
        name: local
        url: "http://127.0.0.1:5000/"
        version: 0
        wsConfig {
          proxy {
            host: "10.1.2.10"
            port: 8080
          }
        }
        includedTheHiveOrganisations: ["ORG1", "ORG2"]
        excludedTheHiveOrganisations: ["ORG3"]
      }
    ]
    ```
Webhook endpoints can be authenticated by adding the setting `auth`. Supported methods are:
 * basic `"auth": { "type": "basic", "username": "foo", "password": "bar" }`
 * bearer `"auth": { "type": "bearer", "key": "foobar" }`
 * key `"auth": { "type": "bearer", "key": "foobar" }`
 * none (default) `"auth": { "type": "none" }`

## 2. Activate webhooks

This action must be done by an organisation admin (with permission `manageConfig`) and requires to run a `curl` command:


```bash
read -p 'Enter the URL of TheHive: ' thehive_url
read -p 'Enter your login: ' thehive_user
read -s -p 'Enter your password: ' thehive_password

curl -XPUT -u$thehive_user:$thehive_password -H 'Content-type: application/json' $thehive_url/api/config/organisation/notification -d '
{
  "value": [
    {
      "delegate": false,
      "trigger": { "name": "AnyEvent"},
      "notifier": { "name": "webhook", "endpoint": "local" }
    }
  ]
}'
```