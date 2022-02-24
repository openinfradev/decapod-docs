# Install

## 사전 환경 준비
* Kubernetes Cluster >= v1.17
* 최소 200Gi 이상 사용 가능한 Kubernetes Storage Class가 있어야 한다.

## Decapod-bootstrap을 이용한 설치
Decapod-bootstrap은 [app-of-apps 패턴](https://argoproj.github.io/argo-cd/operator-manual/cluster-bootstrapping/)을 사용하여 decapod component들을 bootstrap하기 위한 프로젝트이다.
argo-cd를 설치하면서 최초의 meta app을 생성하며, 이 meta app이 실제 사용자 application을 설치해주는 구조로 되어있다.
기본적으로 최소한의 동작을 위해 다음 컴포넌트들을 설치한다.

* argo-cd
* postgresql (argo-workflow가 사용하는 DB)
* argo-workflow

### Fork & clone repository
[Decapod-bootstrap](https://github.com/openinfradev/decapod-bootstrap)을 개인 repository로 fork한 후 clone한다
```
git clone https://github.com/{YOUR_REPO_NAME}/decapod-bootstrap
```

디렉토리 구조는 다음과 같이 되어있다
```
├── README.md
├── argocd-install
│   └── values-override.yaml
├── decapod-apps
│   ├── README.md
│   ├── argo-workflows.yaml
│   ├── db-secret-decapod-db.yaml
│   ├── db-secret-argo.yaml
│   └── postgresql.yaml
└── decapod-projects
    ├── README.md
    └── decapod-controller.yaml
```

* argocd-install 디렉토리는 argo-cd에 bootstrap용 project 및 meta app을 생성하기 위한 argocd helm chart용 value-override file을 포함하고 있다.
* helm chart를 수행하면 argocd에 최초로 decapod-bootstrap이라는 project를 만들고, 해당 프로젝트 아래 다음의 두 application을 생성한다.
    * decapod-projects 는 실제 application용 project를 생성하기 위한 meta app으로서, `decapod-projects`라는 디렉토리를 감시하고 있다가 project manifest 파일이 추가되면 이를 감지하여 argocd project를 생성한다.
    * decapod-apps 는 실제 application을 생성하기 위한 meta app으로서, `decapod-apps` app에서 감시하고 있다가 application manifest 파일이 추가되면 이를 감지하여 argocd application을 생성한다.
```
  additionalApplications:
    - name: decapod-apps
      namespace: argo
      destination:
        namespace: argo
        server: https://kubernetes.default.svc
      project: decapod-bootstrap
      source:
        path: decapod-apps
        repoURL: https://github.com/openinfradev/decapod-bootstrap.git
        targetRevision: HEAD
        directory:
          recurse: true
          jsonnet: {}
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
```

!!! note
    위의 내용은 decapod-bootstrap 프로젝트의 코드이므로 실제 사용시에는 fork한 repository의 주소로 알맞게 치환해서 사용해야 한다!

### Namespace 생성
다음의 두 namespace를 생성한다
```
$ kubectl create ns argo
$ kubectl create ns decapod-db
```

### Install argo-cd
argo-cd 차트와 위에서 준비해놓은 values-override 파일을 이용하여 argo-cd를 설치한다
(chart location: https://artifacthub.io/packages/helm/argo/argo-cd)
```
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm install argo-cd argo/argo-cd --version 3.33.6 -f ./decapod-bootstrap/argocd-install/values-override.yaml -n argo
```
Argo-cd가 설치되면, decapod-projects 및 decapod-apps application이 생성되며, 이 application들이 자신이 바라보고 있는 git repo 상의 디렉토리를 스캔하여 실제 application들을 순차적으로 설치하게 된다.

## 결과 확인
### pod 상태 확인
아래와 같이 모든 pod 의 상태가 running 상태가 됨을 확인한다.
```
$ kubectl get pods -n decapod-db
NAME                      READY   STATUS    RESTARTS   AGE
postgresql-postgresql-0   1/1     Running   0          4m10s

$ kubectl get pods -n argo
NAME                                                           READY   STATUS             RESTARTS   AGE
argo-cd-argocd-application-controller-7bc75f949c-svrnk         1/1     Running            0          5m34s
argo-cd-argocd-dex-server-7bd494f8b5-j5br5                     1/1     Running            0          5m34s
argo-cd-argocd-redis-6f696857c5-zmqfh                          1/1     Running            0          5m34s
argo-cd-argocd-repo-server-545455798b-st5x8                    1/1     Running            0          5m34s
argo-cd-argocd-server-6666cb7689-tswfr                         1/1     Running            0          5m34s
argo-workflows-operator-server-d7df65b99-gpx5q                 0/1     Running            0          4m21s
argo-workflows-operator-workflow-controller-598dfdd565-vrzg9   0/1     Running            0          4m21s
```

### web UI 접속
#### argo-cd
모든 pod 이 Running 상태가 됨을 확인후, 아래 명령어를 통해 argo-cd password 를 확인한다.
```
$ kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

argo-cd nodeport(80)를 통하여 다음과 같이 web UI 에 접속할 수 있어야 한다.

![argo-cd web console](../assets/argo-cd.png)


