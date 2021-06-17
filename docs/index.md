# Overview
## What is Decapod?
Decapod은 Declarative Application Orchestration & Delivery의 약자입니다.  

Decapod은 여러 Kubernetes Cluster 환경에 비슷한 Application Group 들을 배포해야할 때,  
보다 쉬운 `Yaml Documentation`과 `Deployment Pipelining`을 도와줍니다.  


![decapod-flow](assets/decapod-flow.svg)

## Features
* LMA(Logging, Monitoring and Alarm), Service Mesh 등의 [App Group](glossary.md#app_group) 제공
* 사용자의 Kubernetes 환경에 따른 손쉬운 Yaml Customizing 제공
* Application간 Dependency에 따른 Deploy Pipeline

## Projects
* [decapod-flow](https://github.com/openinfradev/decapod-flow) - Argo WorkflowTemplate들의 저장소
* [decapod-base-yaml](https://github.com/openinfradev/decapod-base-yaml) - Kustomize의 base가 되는 base resource yaml
* [decapod-site](https://github.com/openinfradev/decapod-site) - 환경에 맞게 kustomize를 통해 override할 수 있는 site yaml
* [decapod-manifests](https://github.com/openinfradev/decapod-manifests) - base와 site를 합쳐 렌더링한 결과의 저장소