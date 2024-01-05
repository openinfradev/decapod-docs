# Overview
## What is Decapod?
Decapod은 Declarative Application Orchestration & Delivery의 약자로서, 다수의 Kubernetes Cluster 환경에 다양한 Application Group 들을 배포해야할 때, 보다 쉬운 `Yaml Documentation`과 `Deployment Pipelining`을 GitOps 형태로 수행할 수 있게 해준다. 

![decapod-flow](assets/decapod-flow.svg)

## Features
* LMA(Logging, Monitoring and Alarm), Service Mesh 등의 pre-defined [App Group](glossary.md#app_group) 제공
* 사용자의 Kubernetes 환경에 따른 손쉬운 Yaml Customization 제공
* Application간 Dependency를 고려한 순차적/단계적인 Deployment 지원

## Use Cases
SKT Container Platform에서는 Decapod를 기반으로 Cloud Infra Resource, Kubernetes Cluster 및 다양한 서비스들을 GitOps 형태로 배포하고 관리하고 있으며 실제로 다음과 같은 Use Case들에서 Decapod가 활용되고 있다.

* Kubernetes Cluster 생성에 필요한 퍼블릭 클라우드 리소스를 생성 
* Kubernetes Cluster를 다양한 클라우드 환경에 배포 및 관리
* Prometheus, 다양한 Exporter들, Grafana, Loki, Promtail 등의 logging/monitoring tool들을 "LMA"라는 단일 App Group으로 구성하여 일괄 배포 및 관리
* Istio, Jaeger, Kiali, Elastic Search, Service-mesh Portal 등을 "Service-Mesh"라는 단일 App Group으로 구성하여 일괄 배포 및 관리
* Kubeflow, Istio, Knative, Argo Workflow 등을 "AI Serve" 라는 단일 App Group으로 구성하여 일괄 배포 및 관리 

## Projects
* [decapod-flow](https://github.com/openinfradev/decapod-flow) - Argo WorkflowTemplate들의 저장소
* [decapod-base-yaml](https://github.com/openinfradev/decapod-base-yaml) - 생성될 manifest의 base가 되는 base resource yaml
* [decapod-site](https://github.com/openinfradev/decapod-site) - 환경에 맞게 kustomize를 통해 override할 수 있는 site yaml
* [decapod-manifests](https://github.com/openinfradev/decapod-manifests) - base와 site를 합쳐 렌더링한 결과의 저장소
* [decapod-bootstrap](https://github.com/openinfradev/decapod-bootstrap) - 최초 decapod component들을 설치해주는 프로젝트
