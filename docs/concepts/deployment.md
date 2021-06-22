# Deployment
## Overview
Decapod은 Argo Workflow와 Argo CD를 통해 Application Group을 배포한다.  
여러 Application(혹은 Helm Chart)를 배포할 때, 각 Application은 동시에 설치할 수도 있지만 의존성의 문제로 순차적으로 배포해야할 때가 있다.  
Argo Workflow는 이러한 sequential deployment를 가능하게 한다.

또 Argo CD를 통해 Application를 생성하고 배포하여 decapod-manifests의 파일이 변경될 때 automated sync 기능을 통해 자동으로 최신 버전을 유지할 수 있다.

![decapod-flow-details](../assets/decapod-flow-details.svg)

## decapod-flow
_[github link](https://github.com/openinfradev/decapod-flow)_  
Argo Workflow를 통해 LMA, Service Mesh같은 Application Group을 배포한다.  
이 때 필요한 Argo Workflow의 WorkflowTemplate과 그 외의 설정을 `decapod-flow`에 정의하였다.  

### Workflow Templates
| Name | Description | Link |
|------|-------------|------|
|prepare-argocd|Argo CD 인증 정보를 저장하고, project들을 만든다.|[prepare-argocd-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/argo-cd/prepare-argocd-wftpl.yaml)|
|create-application|decapod-manifests repository에 저장된 YAML 파일을 통해 Argo CD에 Application을 생성한다.|[createapp-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/argo-cd/createapp-wftpl.yaml)|
|lma-federation|federation 형상의 LMA를 설치한다. 일반 LMA에서 추가로 Grafana, thanos 등이 설치된다.|[lma-federation-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/lma-federation-wftpl.yaml)|
|lma|일반적인 형상의 LMA을 설치한다.|[lma-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/lma-wftpl.yaml)|
|service-mesh|Service Mesh를 설치한다.|[service-mesh-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/service-mesh-wf.yaml)|
|openstack-infra|OpenStack의 Memcached, Ceph과 같은 Infra에 해당하는 서비스들을 설치한다.|[openstack-infra-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/openstack-infra-wftpl.yaml)|
|openstack-components|OpenStack의 Infra 외의 모든 서비스들을 설치한다.|[openstack-components-wf.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/openstack-components-wf.yaml)|