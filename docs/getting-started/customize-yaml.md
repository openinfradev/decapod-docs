# Customizing
Decapod-site를 배포할 환경에 맞게 수정하여 Argo CD가 사용할 decapod-manifests를 생성한다.

## 사전 준비
* [decapod-site](https://github.com/openinfradev/decapod-site)를 자신의 repo로 fork한다. 
    * [Fork 방법](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
* 렌더링된 [decapod-manifests](https://github.com/openinfradev/decapod-manifests)를 저장할 repository를 생성한다.
    * 예: github.com/<YOUR_REPOSITORY_NAME\>/decapod-manifest.git

## 신규 사이트 생성

    $ git clone https://github.com/<YOUR_REPOSITORY_NAME>/decapod-site.git
    $ cd decapod-site
    $ ./create_site.sh <SITE_NAME>
    Cloning into '.base-yaml'...
    remote: Enumerating objects: 146, done.
    remote: Counting objects: 100% (146/146), done.
    remote: Compressing objects: 100% (106/106), done.
    remote: Total 533 (delta 54), reused 101 (delta 29), pack-reused 387
    Receiving objects: 100% (533/533), 187.43 KiB | 2.53 MiB/s, done.
    Resolving deltas: 100% (186/186), done.

    $ ls <SITE_NAME>
    admin-tools   cloud-console lma           openstack     service-mesh

## 커스터마이징
!!! note
    이 문서는 LMA를 커스터마이징하는 예시이다. service-mesh, openstack 등은 site-values.yaml 내용이 다르므로 각 site-values.yaml 파일 내용을 보고 수정해야한다.

_decapod-site/<SITE_NAME\>/lma/site-values.yaml_:

* 글로벌 변수  
  글로벌 변수는 site-values.yaml 파일 내에서 재사용 가능한 변수이다. 여러 차트에서 동일하게 사용되야하는 변수의 경우 global에 정의하여 `$(GLOBAL_VAR_NAME)`형식으로 사용할 수 있다.
```yaml linenums="6"
global:
  nodeSelector:
    taco-lma: enabled
  clusterName: cluster.local
  storageClassName: taco-storage
  repository: https://openinfradev.github.io/hanu-helm-repo/
  serviceScrapeInterval: 30s
  defaultPassword: password
  defaultUser: taco
  thanosObjstoreSecret: taco-objstore-secret
  thanosPrimaryCluster: false
```

* 차트별 변수  
  IP와 Storage Size와 같은 설정을 환경에 맞게 바꿔준다. 아래는 prometheus의 값을 override하는 예시이다. 
```yaml linenums="25"
- name: prometheus
  override:
    kubeEtcd.endpoints:
    - 172.27.1.222
    prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.storageClassName: $(storageClassName)
    prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage: 10Gi
    prometheus.prometheusSpec.retention: 2d
    prometheus.prometheusSpec.externalLabels.taco_cluster: dev
    prometheus.prometheusSpec.nodeSelector: $(nodeSelector)
```

## 로컬 빌드
!!! note 
    로컬에서 빌드하기 위해서는 `docker` 가 설치되어 있는 환경이어야한다.  
    로컬 빌드는 documentation 검증을 위한 절차로,  
    로컬에서 빌드된 결과물이 decapod-manifests와 같은 github repository에 올라가지 않으면 배포할 수 없다.

`decapod-site/.github/workflows/render-cd.sh` 파일을 사용하여 빌드한다.

```bash
$ cd decapod-site
$ .github/workflows/render-cd.sh <DECAPOD-BASE-BRANCH> <OUTPUT_DIR> <SITE_NAME>
```

실제 배포를 위해서는 빌드된 최종 결과물을 미리 생성해놓은 'decapod-manifest' repository로 push해준다
```
$ cd <YOUR-DECAPOD-SITE-DIRECTORY>
$ cd cd   # 'cd' is directory for output manifests
$ mv ./* <YOUR-DECAPOD-MANIFEST_DIRECTORY>/
$ cd <YOUR-DECAPOD-MANIFEST_DIRECTORY>
$ git commit
$ git push
```

## 자동 빌드
Decapod-site는 Github Action을 통해 빌드를 자동화하였다. Repository에 pull request가 생성되어 main branch에 merge되면, 렌더링된 결과물이 decapod-manifest repo로 자동으로 push 되며, 자세한 내용은 [여기](https://github.com/openinfradev/decapod-site/blob/main/.github/workflows/merge_main.yml)를 참고할 수 있다.
