# Troubleshooting

## Workflow Error 발생 시
1. Argo Workflow UI에서 에러를 확인한다.
  [![](./assets/decapod-log.png)](./assets/decapod-log.png)

2. Argo CD UI를 통해 에러를 확인한다.
  
3. Kubectl CLI를 통해 에러를 확인한다.
```sh
kubectl get po -nlma
kubectl describe po <문제되는 POD의 이름>
```