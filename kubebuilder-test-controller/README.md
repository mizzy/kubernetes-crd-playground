# Kubebuilder Test Controller

Ref [Kubebuilderを使ってみる - Toku's Blog](https://cstoku.io/posts/2018/kubebuilder-intro/)


```
$ kubebuilder init --domain mizzy.org --license apache2 --owner mizzy
$ kubebuilder create api --group trial --version v1alpha1 --kind EchoField
```


```
$ make install
CRD manifests generated under '/Users/mizzy/src/github.com/mizzy/kubernetes-crd-playground/kubebuilder-test-controller/config/crds' 
RBAC manifests generated under '/Users/mizzy/src/github.com/mizzy/kubernetes-crd-playground/kubebuilder-test-controller/config/rbac' 
kubectl apply -f config/crds
customresourcedefinition.apiextensions.k8s.io/echofields.trial.mizzy.org created
```

```
$ make run
```

On another terminal:

```
$ kubectl apply -f config/samples/trial_v1alpha1_echofield.yaml
echofield.trial.mizzy.org/echofield-sample created

```

```
$ kubectl get echofield
NAME               AGE
echofield-sample   13m
```

```
$ kubectl get all
NAME                                               READY   STATUS    RESTARTS   AGE
pod/echofield-sample-deployment-7dfcdc84c6-xc4fz   1/1     Running   0          112m

NAME                                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echofield-sample-deployment   1         1         1            1           112m

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echofield-sample-deployment-7dfcdc84c6   1         1         1       112m
```
