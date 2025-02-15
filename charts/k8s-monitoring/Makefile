SHELL := /bin/bash

values.schema.json: values.yaml schema-mods/enums-and-types.json schema-mods/required-values.json
	docker run -it --rm --platform linux/amd64 -v "$(shell pwd):/chart" --entrypoint sh alpine/helm \
		--command -c 'helm plugin install https://github.com/karuppiah7890/helm-schema-gen.git && \
			helm schema-gen /chart/values.yaml > /chart/values.schema.generated.json'

	jq --indent 4 -s '.[0] * .[1] * .[2]' \
		<(jq --indent 4 'del(.properties["grafana-agent"]) \
			| del(.properties["grafana-agent-events"]) \
			| del(.properties["grafana-agent-logs"]) \
			| del(.properties["kube-state-metrics"].properties.autosharding) \
			| del(.properties["kube-state-metrics"].properties.nodeSelector) \
			| del(.properties["kube-state-metrics"].properties.prometheusScrape) \
			| del(.properties["kube-state-metrics"].properties.updateStrategy) \
			| del(.properties["prometheus-node-exporter"].properties.nodeSelector) \
			| del(.properties["prometheus-node-exporter"].properties.service) \
			| del(.properties["prometheus-windows-exporter"].properties.config) \
			| del(.properties.opencost.properties.opencost)' values.schema.generated.json) \
		schema-mods/enums-and-types.json \
		schema-mods/required-values.json \
		> values.schema.json

	rm values.schema.generated.json

HAS_HELM_DOCS := $(shell command -v helm-docs;)


README.md: README.md.gotmpl values.yaml Chart.yaml
ifdef HAS_HELM_DOCS
	helm-docs
else
	docker run --rm --volume "$(shell pwd):/helm-docs" -u $(shell id -u) jnorwood/helm-docs:latest
endif

docs/RBAC.md: ../../examples/default-values/output.yaml ../../examples/openshift-compatible/output.yaml
	echo "# RBAC Rules for the Kubernetes Monitoring Helm Chart" > docs/RBAC.md
	echo "Several components have specialized RBAC rules to perform their work. This document will list the components, and their RBAC definitions." >> docs/RBAC.md

	echo "## Grafana Agent" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-grafana-agent") | select(.kind=="ClusterRole") | .rules' ../../examples/default-values/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md

	echo "## Grafana Agent Events" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-grafana-agent-events") | select(.kind=="ClusterRole") | .rules' ../../examples/default-values/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md

	echo "## Grafana Agent Logs" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-grafana-agent-logs") | select(.kind=="ClusterRole") | .rules' ../../examples/default-values/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md
	echo "When deploying to an OpenShift cluster, these extra rules are added to enable access to special a SecurityContextConstraint:" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-grafana-agent-logs-scc") | select(.kind=="ClusterRole") | .rules' ../../examples/openshift-compatible/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md

	echo "## Kube State Metrics" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-kube-state-metrics") | select(.kind=="ClusterRole") | .rules' ../../examples/default-values/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md

	echo "## OpenCost" >> docs/RBAC.md
	echo '```yaml' >> docs/RBAC.md
	yq -y 'select(.metadata.name=="k8smon-opencost") | select(.kind=="ClusterRole") | .rules' ../../examples/default-values/output.yaml >> docs/RBAC.md
	echo '```' >> docs/RBAC.md

build-image: test/Dockerfile test/config-analysis.sh test/query-test.sh
	docker build --tag ghcr.io/grafana/k8s-monitoring-test:$(shell yq -r '.version' Chart.yaml) test

push-image:
	docker push ghcr.io/grafana/k8s-monitoring-test:$(shell yq -r '.version' Chart.yaml)
