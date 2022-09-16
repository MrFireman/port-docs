---
sidebar_position: 1
---

# Local debugging webhook

In this guide, we will show how to debug Webhook `Action` that is sent via Port.

## Prerequisites

- A Port API `CLIENT_ID` and `CLIENT_SECRET`
- Python & PIP installed
- Nodejs

Interaction with Port will be primarily conducted using the API in this example, but everything can also be performed using the web UI as well.

## Scenario

Whenever the amount of free storage gets too low, create a new execution function that frees up or extends the storage in your VM. Let’s learn how to do that, and what are Port’s change log capabilities.

## Creating the VM blueprint

Let’s configure a `VM` Blueprint, its base structure is:

```json showLineNumbers
{
  "identifier": "vm",
  "title": "VM",
  "icon": "Server",
  "schema": {
    "properties": {
      "region": {
        "title": "Region",
        "type": "string",
        "description": "Region of the VM"
      },
      "cpu_cores": {
        "title": "CPU Cores",
        "type": "number",
        "description": "Number of allocated CPU cores"
      },
      "memory_size": {
        "title": "Memory Size ",
        "type": "number",
        "description": "Amount of allocated memory (GB)"
      },
      "storage_size": {
        "title": "Storage Size",
        "type": "number",
        "description": "Amount of allocated storage (GB)"
      },
      "free_storage": {
        "title": "Free Storage",
        "type": "number",
        "description": "Amount of free storage (GB)"
      },
      "deployed": {
        "title": "Deploy Status",
        "type": "string",
        "description": "The deployment status of this VM"
      }
    },
    "required": []
  },
  "disableEditing": false,
  "enableResponsibleTeamEdit": false,
  "disabledProperties": [],
  "disabledRelations": [],
  "formulaProperties": {}
}
```

Below you can see the `python` code to create this Blueprint (remember to insert your `CLIENT_ID` and `CLIENT_SECRET` in order to get an access token)

<details>
<summary>Click here to see the code</summary>

```python showLineNumbers
import requests

CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

API_URL = 'https://api.getport.io/v1'

credentials = {'clientId': CLIENT_ID, 'clientSecret': CLIENT_SECRET}

token_response = requests.post(f'{API_URL}/auth/access_token', json=credentials)

access_token = token_response.json()['accessToken']

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint = {
    "identifier": "vm",
    "title": "VM",
    "icon": "Server",
    "schema": {
        "properties": {
            "region": {
                "title": "Region",
                "type": "string",
                "description": "Region of the VM"
            },
            "cpu_cores": {
                "title": "CPU Cores",
                "type": "number",
                "description": "Number of allocated CPU cores"
            },
            "memory_size": {
                "title": "Memory Size ",
                "type": "number",
                "description": "Amount of allocated memory (GB)"
            },
            "storage_size": {
                "title": "Storage Size",
                "type": "number",
                "description": "Amount of allocated storage (GB)"
            },
            "free_storage": {
                "title": "Free Storage",
                "type": "number",
                "description": "Amount of free storage"
            },
            "deployed": {
                "title": "Deploy Status",
                "type": "string",
                "description": "The deployment status of this VM"
            }
        },
        "required": []
    },
    "disableEditing": False,
    "enableResponsibleTeamEdit": False,
    "disabledProperties": [],
    "disabledRelations": [],
    "formulaProperties": {}
}

response = requests.post(f'{API_URL}/blueprints', json=blueprint, headers=headers)

print(response.json())
```

</details>

## Creating the VM CREATE action

In order to be able to forward webhooks to the localhost, we will use [smee.io](https://smee.io/), Just click on `Start new channel` and copy the provided `Webhook Proxy URL` it should look similar to this: `https://smee.io/b1iO4C4ZGNYmiVL5`.

Now let’s configure a Self-Service Action. You will add a `CREATE` action that will be triggered every time a developer creates a new VM entity, the Self-Service Action will trigger your Lambda.
Here is the action JSON:

```json showLineNumbers
{
  "identifier": "create_vm",
  "title": "Create VM",
  "icon": "Server",
  "description": "Create a new VM in cloud provider infrastructure",
  "trigger": "CREATE",
  "invocationMethod": { "type": "WEBHOOK", "url": "<PUT SMEE URL HERE>" },
  "userInputs": {
    "properties": {
      "title": {
        "type": "string",
        "title": "Title of the new VM"
      },
      "cpu": {
        "type": "number",
        "title": "Number of CPU cores"
      },
      "memory": {
        "type": "number",
        "title": "Size of memory"
      },
      "storage": {
        "type": "number",
        "title": "Size of storage"
      },
      "region": {
        "type": "string",
        "title": "Deployment region",
        "enum": ["eu-west-1", "eu-west-2", "us-west-1", "us-east-1"]
      }
    },
    "required": ["cpu", "memory", "storage", "region"]
  }
}
```

Below you can see the `python` code to create this action (remember to insert your `CLIENT_ID` and `CLIENT_SECRET` in order to get an access token).

:::note Specifying the target blueprint
Note how the `vm` Blueprint identifier is used to add the action to the new Blueprint
:::

<details>
<summary>Click here to see code</summary>

```python showLineNumbers
import requests

CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

API_URL = 'https://api.getport.io/v1'

credentials = {'clientId': CLIENT_ID, 'clientSecret': CLIENT_SECRET}

token_response = requests.post(f'{API_URL}/auth/access_token', json=credentials)

access_token = token_response.json()['accessToken']

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint_identifier = 'vm'

action = {
    'identifier': 'create_vm',
    'title': 'Create VM',
    'icon': 'Server',
    'description': 'Create a new VM in cloud provider infrastructure',
    'trigger': 'CREATE',
    "invocationMethod": { 'type': 'WEBHOOK', 'url': '<PUT SMEE URL HERE>' },
    'userInputs': {
        'properties': {
            'title': {
                'type': 'string',
                'title': 'Title of the new VM'
            },
            'cpu': {
                'type': 'number',
                'title': 'Number of CPU cores'
            },
            'memory': {
                'type': 'number',
                'title': 'Size of memory'
            },
            'storage': {
                'type': 'number',pysmee
                'title': 'Size of storage'
            },
            'region': {
                'type': 'string',
                'title': 'Deployment region',
                'enum': ['eu-west-1', 'eu-west-2', 'us-west-1', 'us-east-1']
            }
        },
        'required': [
            'cpu', 'memory', 'storage', 'region'
        ]
    }
}

response = requests.post(f'{API_URL}/blueprints/{blueprint_identifier}/actions', json=action, headers=headers)

print(response.json())
```

</details>

## Forwarding events to localhost

Now let's install the Smee client to be able to forward the events to localhost, we will use pysmee to achieve that

```bash
pip install pysmee
```

Now lets use it to forward the event, for example

```bash
pysmee forward https://smee.io/b1iO4C4ZGNYmiVL5 http://localhost:3000/webhooks
```

You should see a log line output like this

```bash
[2022-09-15 13:59:39,462 MainThread] INFO: Forwarding https://smee.io/b1iO4C4ZGNYmiVL5 to http://localhost:3000/webhooks
```

## Creating small example server in Nodejs

Now because we are forwarding events to the localhost, all we need to do is to create a small server that will listed to `POST` requests that are being sent to the /webhooks route.

:::info
In this example we are using Nodejs and express but you can use any language and framework you want
:::

Create a folder and run the following in it

```bash
npm init -y
npm install express
```

Inside this folder create an `index.js` file and copy these contents

```js
const { createHmac } = require("crypto");
const express = require("express");
const app = express();
const port = 3000;

app.post("/webhooks", (request, response) => {
  // This part is only to verify that port sent the data and not anybody else
  const signed = createHmac("sha256", "<CLIENT_SECRET>")
    .update(
      `${request.headers["x-port-timestamp"]}.${JSON.stringify(request.body)}`
    )
    .digest("base64");

  if (signed !== request.headers["x-port-signature"].split(",")[1]) {
    throw new Error("Invalid singature");
  }

  // You can put any custom logic here
  console.log("Success!");

  response.send("Hello World!");
});

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
});
```

now let's run the server

```bash
node index.js
```

## Triggering the action

Login to port and go to the VM page and trigger the action via the

![Port Kafka Architecture](../../../../static/img/tutorial/CreateVMDropdown.png)

Fill the wanted details and click on `Create`

![Port Kafka Architecture](../../../../static/img/tutorial/CreateVMExecution.png)

And that's it, you should see a `Success!` output

![Port Kafka Architecture](../../../../static/img/tutorial/HelloWorldLog.png)