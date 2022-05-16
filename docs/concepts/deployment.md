# Deployment
## Overview
Decapod는 Argo Workflow와 Argo CD를 통해 Application Group을 배포한다.  
여러 Application(혹은 Helm Chart)를 배포할 때, 각 Application들은 동시에 설치될 수도 있지만 의존성의 문제로 순차적으로 배포해야할 때가 있다.  
Argo Workflow는 이러한 sequential deployment를 가능하게 한다.

또 Argo CD를 통해 Application들이 배포되므로, 향후 decapod-manifests 파일이 변경될 때 automated sync 기능을 통해 자동으로 변경 사항이 적용되어 항상 최신 버전을 유지할 수 있다.

![decapod-flow-details](../assets/decapod-flow-details.svg)

## decapod-flow
_[github link](https://github.com/openinfradev/decapod-flow)_  
Argo Workflow를 통해 LMA, Service Mesh 등의 Application Group을 배포한다.  
이 때 필요한 Argo Workflow Template과 그 외의 설정을 `decapod-flow`에 정의하였다.  

### Workflow Templates
| Name | Description | Link |
|------|-------------|------|
|prepare-argocd|Argo CD 인증 정보를 저장하고, project들을 만든다.|[prepare-argocd-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/argo-cd/prepare-argocd-wftpl.yaml)|
|create-application|decapod-manifests repository에 저장된 YAML 파일을 통해 Argo CD에 Application을 생성한다.|[createapp-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/argo-cd/createapp-wftpl.yaml)|
|delete-apps|Argo CD Application을 삭제한다.|[delete-apps-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/argo-cd/delete-apps-wftpl.yaml)|
|lma-federation|federation 형상의 LMA를 설치한다. Grafana, thanos 등이 포함된다.|[lma-federation-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/lma-federation-wftpl.yaml)|
|service-mesh|Service Mesh를 설치한다.|[service-mesh-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/service-mesh-wf.yaml)|
|openstack-infra|OpenStack의 Memcached, Ceph과 같은 Infra에 해당하는 서비스들을 설치한다.|[openstack-infra-wftpl.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/openstack-infra-wftpl.yaml)|
|openstack-components|OpenStack의 Infra 외의 모든 서비스들을 설치한다.|[openstack-components-wf.yaml](https://github.com/openinfradev/decapod-flow/blob/main/templates/decapod-apps/openstack-components-wf.yaml)|

위의 template 외에 "remove-APPGROUP" template의 경우 해당 app group을 삭제하는 역할을 수행한다.
