# Docker Infrastructure

## Commands:
Notes:
before using this commands please change `$prject_root` to real project root,
change `{{servie_name}}` to your sevice name, change `prject_name` to projet name

Build docker image
```sh
docker build -f $prject_root/../docker_infrastructure/dockerfiles/ruby/3.2.4/dev/Dockerfile --platform=linux/amd64 -t $prject_name:dev .
```
Run docker container

```sh
docker exec -u=0 -it $(docker ps | grep {{servie_name}} | awk '{ print $1 }') /bin/bash
```

## Steps
1. Copy files from template folder
```sh
cp templates/.env-template ./compose_files/.env
cp templates/elastic-agent-template.yml volumes/apm_server/config/elastic-agent.yml
cp templates/elastic-agent-template.yml volumes/apm_server/config/elastic-agent.yml
```

2. Replace `REPLACE_ME` with real value

3. Run ElasticSearch Kibana
```bash
docker compose -f compose_files/docker-compose.ek-stack.yml up
```
4. Configure Fleet Agent policies and Flet Server via Kibana
    1. Login to kibana
    ```
    http://0.0.0.0:5601/
    ```

    2. Go Fleet Policies
    ```
    tab[Agent policies]
    btn[Create agent policy]
    name: Fleet Server Policy

    btn[Preview API request]
    ```

    3. Create Fleet Policy

    Open in Console

    ```sh
    POST kbn:/api/fleet/agent_policies
    {
      "name": "Fleet Server Policy",
      "id": "fleet-server-policy",
      "description": "Static agent policy for Fleet Server",
      "namespace": "default",
      "monitoring_enabled": [
        "logs",
        "metrics"
      ],
      "inactivity_timeout": 1209600,
      "is_protected": false
    }
    ```
    ```
    btn[Send request]
    ```

    4. Add integration to Fleet Policy
    Open policy
    ```sh
    http://0.0.0.0:5601/app/fleet/policies/fleet-server-policy
    ```
    ```
    btn[Add integration]
    ```
    Fleet Server

    Add Fleet Server

    ```
    btn[Save and continue]
    ```


    5. Create agent policy
    ```
    tab[Agent policies]
    btn[Create agent policy]
    name: Agent Policy APM Server

    btn[Preview API request]
    btn[Open in Console]
    ```

    ```sh
    POST kbn:/api/fleet/agent_policies
    {
      "name": "Agent Policy APM Server",
      "id": "agent-policy-apm-server",
      "description": "Static agent policy for the APM Server integration",
      "namespace": "default",
      "monitoring_enabled": [
        "logs",
        "metrics"
      ],
      "inactivity_timeout": 1209600,
      "is_protected": false
    }
    ```
    ```
    btn[Send request]
    ```
    6. Add integration Elastic APM
    Open policy
    ```
    http://0.0.0.0:5601/app/fleet/policies/agent-policy-apm-server
    ```
    ```
    btn[Add integration]
    ```
    Search `APM` and click on it

    ```
    tab[Elastic APM in Fleet]

    (bellow)
    tab[OpenTelemetry]
    link[Get started with fleet]
    btn[Add Elastic APM]
    ```
    ```
    Host: 0.0.0.0:8200
    URL: http://apm-server:8200
    ```

    Agent authorization
    ```
    Secret token: generated_token
    ```
    ```
    Where to add this integration?
    tab[Existing hosts]
    Agent policies: Agent Policy APM Server

    btn[Save and continue]
    ```

```
Management -> Fleet -> Settings
http://0.0.0.0:5601/app/fleet/settings
```

```
Add Fleet Server

Name: Fleet Server
URL: https://fleet-server:8220
```

```
Outputs
Add output

Hosts: https://es01:9200
Advanced YAML configuration

"ssl.verification_mode": "none"
```
btn[Save and apply settings]


Go to agents
```
http://0.0.0.0:5601/app/fleet/agents
```
btn[Add Fleet Server]
tab[Advanced]

Generate a service token
btn[Generate service token]

Copy token to env file
```sh
# .env
FLEET_SERVER_SERVICE_TOKEN=$GENERATED_TOKEN
```

Run Fleet
```
docker compose -f compose_files/docker-compose.fleet.yml up
```

Copy Secret for  Agent Policy APM Server
Fleet
tab[Enrollment tokens]

```
http://0.0.0.0:5601/app/fleet/enrollment-tokens
```

```sh
# .env
FLEET_ENROLLMENT_TOKEN=$SECRET_FOR_POLICY
```

Fleet
btn[Add agent]

```sh
docker compose -f compose_files/docker-compose.apm-server.yml up
```

### elasticsearch sample
https://github.com/deviantony/docker-elk/blob/main/docker-compose.yml
