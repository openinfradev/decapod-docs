# Documentation
## Overview
Decapod의 Documentation 기능은 반복적이고 복잡한 Yaml 파일 작성을 줄여준다.  
다수의 Helm Chart를 하나의 Application Group으로 묶어서 관리하고 하나의 파일에서 모든 차트의 value를 override할 수 있다.  

모든 YAML 설정을 Git Repository에 저장하여 배포하는 GitOps 체계이다.  

YAML 설정을 Base와 Site로 나뉘어 관리하며 Base는 Application Group에 대한 Default YAML, 즉 각 환경과 독립적인 공통적인 설정값들을 정의한다.  
Base의 값을 내 환경에 맞게 변경하고자 한다면 Site에서 Value Override를 할 수 있다.  
이는 Helm Chart에서 `Template`과 `values.yaml` 같은 역할이라 볼 수 있다.

![decapod-documentation](../assets/decapod-documentation.svg)

## decapod-base-yaml 
_[github link](https://github.com/openinfradev/decapod-base-yaml)_  

kustomize 규격에 맞게 base resources file들을 미리 정의해 놓은 곳이다.  
여러 Helm chart를 묶어서 LMA, Service Mesh 등의 Application Group으로 정의하였으며,  
환경별로 주로 수정할 value들을 모아놓은 `site-values.yaml` 을 제공한다.

### Layout
    service-mesh
    ├── base
    │   ├── kustomization.yaml
    │   ├── resources.yaml
    │   └── site-values.yaml
    ├── image
    │   └── image-values.yaml
    lma
    ├── base
    │   ├── kustomization.yaml
    │   ├── resources.yaml
    │   └── site-values.yaml
    └── image
        └── image-values.yaml

최상단 Directory는 Application Group으로 구분된다.  
각 Application Group 안에는 base와 image directory가 있다.
각 디렉토리의 역할은 다음과 같다.  

  * base - 전체 application 정보를 담은 `resources.yaml`과 value override 값을 담고 있는 `site-values.yaml`이 정의되어 있다.
  * image  - Air Gap 환경에서 배포해야할 때, 모든 docker image 파일을 내부 docker registry를 바라보도록 바꿔야한다. 이를 위해 미리 모든 image 값을 override할 수 있도록 정의해놓은 overlay 파일이다.


### Application Group
_[github link](https://github.com/openinfradev/decapod-site)_  

현재 decapod-base-yaml에서 제공하는 Application Group은 다음과 같다.  

1. LMA (Logging, Monitoring and Alarm)
2. Service Mesh
3. OpenStack


## decapod-site
decapod-base-yaml에 정의된 Application Group을 배포 환경에 맞게 customize하기 위한 설정을 저장하는 곳이다.
사용자들이 올린 PR이 decapod-site main branch로 merge되면 github action을 통해 자동으로 kustomize build 명령이 수행되어 결과물이 decapod-manifests repository에 저장된다.
### Layout
    ├── 사이트명
    │   ├── application이름
    │   │   ├── kustomization.yaml
    │   │   └── site-values.yaml
    ├── decapod-reference
    │   ├── lma
    │   │   ├── kustomization.yaml
    │   │   └── site-values.yaml
    │   ├── openstack
    │   │   ├── kustomization.yaml
    │   │   └── site-values.yaml
    │   └── service-mesh
    │       ├── kustomization.yaml
    └── decapod-reference-offline
        ├── lma
        │   ├── image-values.yaml
        │   ├── kustomization.yaml
        │   └── site-values.yaml
        └── service-mesh
            ├── image-values.yaml
            ├── kustomization.yaml
            └── site-values.yaml

최상단 디렉토리는 사이트(환경)으로 구분된다. 온라인 환경을 위한 sample site는 `decapod-reference` 이고,
오프라인(Air-gapped) 환경을 위한 sample site는 `decapod-reference-offline` 이다.

## decapod-manifests
_[github link](https://github.com/openinfradev/decapod-manifests)_  
decapod-base-yaml과 decapod-site로부터 kustomize build된 **결과물**이 저장되는 곳이다.  
decapod-site와 동일한 directory 구조를 가지지만, 최종 output의 형태는 Helm chart로부터 생성된 plaintext yaml이다.  
## Example
base(1) + site(2) => manifests(3)

1. decapod-base-yaml/lma/base/resources.yaml:

        apiVersion: helm.fluxcd.io/v1
        kind: HelmRelease
        metadata:
        name: elasticsearch-operator
        spec:
        chart:
          repository: https://openinfradev.github.io/helm-repo
          name: elasticsearch-operator
          version: 1.0.3
        releaseName: elasticsearch-operator
        targetNamespace: elastic-system
        values:
          elasticsearchOperator:
              nodeSelector: {} # TO_BE_FIXED

2. decapod-site/dev/lma/site-values.yaml:

        apiVersion: openinfradev.github.com/v1
        kind: HelmValuesTransformer
        metadata:
        name: site

        global:
        nodeSelector:
          taco-lma: enabled

        charts:
        - name: elasticsearch-operator
        override:
          elasticsearchOperator.nodeSelector: $(nodeSelector)


3. decapod-site/dev/lma/lma-manifest.yaml:

        apiVersion: helm.fluxcd.io/v1
        kind: HelmRelease
        metadata:
        name: elasticsearch-operator
        spec:
        chart:
          repository: https://openinfradev.github.io/helm-repo
          name: elasticsearch-operator
          version: 1.0.3
        releaseName: elasticsearch-operator
        targetNamespace: elastic-system
        values:
          elasticsearchOperator:
              nodeSelector:
              taco-lma: enabled
