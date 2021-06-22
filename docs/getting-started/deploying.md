# Deploying

## 사전 준비
* 접속 가능한 Argo Workflow와 Argo CD, argo CLI가 설치되어 있어야 한다.

* Argo Workflow에 decapod-flow에서 제공하는 `WorkflowTemplate` 이 생성되어 있어야 한다.  
[이곳](./install.md#)에서 tacoplay를 통해서 설치했다면 WorkflowTemplate이 생성되어 있다.  
만약, 수동 설치를 하였다면 아래와 같이 argo template을 생성해준다.
```bash
git clone https://github.com/openinfradev/decapod-flow.git
cd decapod-flow/templates
argo template create argo-cd/prepare-argocd-wftpl.yaml
argo template create argo-cd/createapp-wftpl.yaml
argo template create decapod-apps/lma-federation-wftpl.yaml
argo template create decapod-apps/lma-wftpl.yaml
argo template create decapod-apps/openstack-components-wftpl.yaml
argo template create decapod-apps/openstack-infra-wftpl.yaml
argo template create decapod-apps/service-mesh-wf.yaml
``` 

## 배포
!!! note
    이 문서에서는 LMA 배포에 대해서만 다룬다. 
### Argo CLI로 배포
1. 최초 한 번만 prepare-argocd를 실행한다. parameter는 환경에 맞게 수정한다.  
  ```sh
  argo submit -n argo --from wftmpl/prepare-argocd \
    -p argo_username="admin" # argo cd의 유저 아이디 \
    -p argo_password="password" # argo cd 유저의 패스워드 \
    -p argo_server="decapod-argocd-server.argo.svc:80" # argo cd 주소

  argo ls -n argo # 생성된 workflow가 완료될 때까지 기다린다.
  ```

2. lma 배포
  ```sh
  argo submit -n argo --from wftmpl/lma-federation \
    -p site_name="사이트명" # decapod-manifests의 사이트 디렉토리명과 일치해야한다. \
    -p app_name="lma" \
    -p repository_url="https://github.com/openinfradev/decapod-manifests" # decapod-manifests repository 주소

  argo ls -n argo # 생성된 workflow가 완료될 때까지 기다린다.
  ```

3. 배포 확인  
   
    LMA가 정상적으로 배포되면 다음과 같은 UI에 정상적으로 접근할 수 있어야한다.  

    * Prometheus 30008 NodePort
    * Grafana 30009 NodePort
    * Kibana 30001 NodePort
  
    다음과 같은 명령어로 모든 pod의 상태를 확인할 수 있다.
    ```sh
    kubectl get pod -n lma
    ```