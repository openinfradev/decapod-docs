# Deploying

## 사전 준비
* argo CLI 가 설치되어 있어야 한다.
    * [argo CLI 설치](https://github.com/argoproj/argo-workflows/releases) 

* Argo Workflow에 decapod-flow에서 제공하는 `WorkflowTemplate` 을 아래와 같이 생성한다. 각각은 decapod에서 제공하는 pre-defined workflow 템플릿이다.
```bash
git clone https://github.com/openinfradev/decapod-flow.git
cd decapod-flow/templates

# argo 를 위한 rbac manifest 를 적용한다.
kubectl apply -f argo-additional-rbac.yaml

argo template create argo-cd/prepare-argocd-wftpl.yaml
argo template create argo-cd/createapp-wftpl.yaml

argo template create decapod-apps/lma-uniformed-wftpl.yaml
argo template create decapod-apps/openstack-components-wf.yaml
argo template create decapod-apps/openstack-infra-wftpl.yaml
argo template create decapod-apps/service-mesh-wf.yaml
``` 

## 배포
!!! note
    이 문서에서는 LMA (Logging,Monitoring,Allerting) 배포에 대해서만 다룬다. 
### Argo CLI로 배포
1. 최초 한 번만 prepare-argocd를 실행한다. parameter는 환경에 맞게 수정한다.  
  ```sh
  # 아래 명령어를 통하여 argo-cd password 를 확인할 수 있다.
  kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

  argo submit -n argo --from wftmpl/prepare-argocd \
    -p argo_username="admin" \
    -p argo_password="PASSWORD" # 위에서 확인된 패스워드를 입력한다. \
    -p argo_server="argo-cd-argocd-server.argo.svc:80"

  argo list -n argo # 생성된 workflow가 완료될 때까지 기다린다.
  ```

2. lma 배포
  ```sh
  argo submit -n argo --from wftmpl/lma-federation \
    -p site_name="YOUR_REPOSITORY_NAME" # decapod-manifests의 사이트 디렉토리명과 반드시 일치해야한다. \
    -p app_name="lma" \
    -p repository_url="https://github.com/openinfradev/decapod-manifests" # decapod-manifests repository 주소

  argo list -n argo # 생성된 workflow가 완료될 때까지 기다린다.
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

    