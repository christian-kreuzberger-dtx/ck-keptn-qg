# Keptn Self-Monitoring using Dynatrace and Keptn 0.7.3

This tutorial adds a new single-stage project to Keptn for self-monitoring Keptn-services.


1. Pre-Requesit: Keptn installation on Kubernetes, fully monitored by Dynatrace; Also, dynatrace-sli-service needs to be installed as a Keptn-service.

2. Create Auto Tagging rules 
  * if Kubernetes namespace=keptn -> Add Tag keptn_selfmon
  * if Kubernetes namespace=keptn -> keptn_selfmon_service: {ProcessGroup:KubernetesContainerName}


3. Create Keptn Project

Download https://raw.githubusercontent.com/keptn/examples/release-0.7.3/onboarding-carts/shipyard-quality-gates.yaml

```
PROJECT=keptn-selfmonitoring
keptn create project $PROJECT --shipyard=./shipyard-quality-gates.yaml

```

4. Configure lighthouse service to use dynatrace-sli-service for this project
```
kubectl create configmap -n ${KEPTN_NAMESPACE} lighthouse-config-${PROJECT} --from-literal=sli-provider=dynatrace
```


5. Add Dynatrace SLI Yaml for all services and stages in the project 

Download https://raw.githubusercontent.com/christian-kreuzberger-dtx/ck-keptn-qg/master/dynatrace/sli.yaml

```
keptn add-resource --project=$PROJECT --resource=sli.yaml --resourceUri=dynatrace/sli.yaml
```

6. Create services

```
SERVICES=("bridge" "eventbroker-go" "configuration-service" "mongodb-datastore" "gatekeeper-service" "remediation-service" "lighthouse-service" "statistics-service" "gatekeeper-service" "dynatrace-sli-service" "jmeter-service" "dynatrace-service" "api-service" "api-gateway-nginx")

for SERVICE in "${SERVICES[@]}"
do
    keptn create service $SERVICE --project=$PROJECT
done
```


7. Add SLOs to all services

Note: Right now there is no way to create an slo for all services automatically (afaik).

Download https://raw.githubusercontent.com/christian-kreuzberger-dtx/ck-keptn-qg/hardening/bridge/slo.yaml

```
for SERVICE in "${SERVICES[@]}"
do
    keptn add-resource --project=$PROJECT --service=$SERVICE --stage=hardening --resource=slo.yaml --resourceUri=slo.yaml
done
```


8. Start evaluation for all services

```
for SERVICE in "${SERVICES[@]}"
do
    keptn send event start-evaluation --project=$PROJECT --service=$SERVICE --stage=hardening
done

```

Or loop it:

```

while true; do 
	for SERVICE in "${SERVICES[@]}"
          do
                keptn send event start-evaluation --project=$PROJECT --service=$SERVICE --stage=hardening
                sleep 1
	done

	sleep 300
done

```

9. Watch output in Keptn Bridge


