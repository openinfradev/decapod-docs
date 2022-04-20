# Deploying applications

## 사전 준비
* argo CLI 가 설치되어 있어야 한다.
    * [argo CLI 설치](https://github.com/argoproj/argo-workflows/releases) 

* decapod-flow에서 제공하는 `WorkflowTemplate` 을 아래와 같이 Argo Workflow Server에 등록한다.  
  각각은 decapod에서 제공하는 pre-defined workflow 템플릿이다.
```bash
git clone https://github.com/openinfradev/decapod-flow.git
cd decapod-flow/templates

# argo 를 위한 rbac manifest 를 적용한다.
kubectl apply -f argo-additional-rbac.yaml

# workflow template을 argo workflow server에 등록한다.
kubectl apply -f ./argo-cd/
kubectl apply -f ./decapod-apps/
``` 

## 배포
!!! note
    이 문서에서는 LMA (Logging/Monitoring/Allerting) 배포에 대해서만 다룬다. 
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
  현재 LMA app group 중 logging componet의 경우 [loki](https://grafana.com/docs/loki/latest/)와 [efk](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) 중에서 선택할 수 있으며, 간단한 테스트 목적의 경우 좀더 lightweight한 'loki'를 권장한다.
  ```sh
  argo submit -n argo --from wftmpl/lma-federation \
    -p site_name="사이트명" # decapod-manifests의 사이트 디렉토리명과 반드시 일치해야한다. \
    -p logging_component="loki" \
    -p manifest_repo_url="https://github.com/{YOUR_REPO_NAME}/decapod-manifests" # decapod-manifests repository 주소

  argo list -n argo # 생성된 workflow가 완료될 때까지 기다린다.
  ```

3. 배포 확인  
  LMA가 정상적으로 배포되면 다음과 같이 pod 및 service 상태를 확인 후 브라우져를 통해 서비스에 접속해본다.  
  실제 service의 NodePort 번호 등은 설정에 따라 조금씩 다를 수 있다.
  ```sh
  $ kubectl get pod -n lma
  NAME                                            READY   STATUS      RESTARTS       AGE
  prometheus-lma-prometheus-0                     3/3     Running     0              18d
  loki-0                                          1/1     Running     0              20h
  grafana-7d5546bb4b-hvkqg                        3/3     Running     0              18d
  promtail-fctfs                                  1/1     Running     0              18d
  ...

  $ kubectl get svc -n lma
  NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                           AGE
  grafana                        ClusterIP   10.233.59.153   <none>        80/TCP                            18d
  lma-prometheus                 NodePort    10.233.45.183   <none>        9090:30008/TCP                    18d
  loki                           ClusterIP   10.233.62.51    <none>        3100/TCP                          18d
  ...
  ```
