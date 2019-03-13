# Kubebuilder Test Controller

Ref [Kubebuilderを使ってみる - Toku's Blog](https://cstoku.io/posts/2018/kubebuilder-intro/)

## Run scffold controller

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



## Customize scaffold controller

```diff
diff --git a/kubebuilder-test-controller/pkg/apis/trial/v1alpha1/echofield_types.go b/kubebuilder-test-controller/pkg/apis/trial/v1alpha1/echofield_types.go
index 821bd50..b92fb18 100644
--- a/kubebuilder-test-controller/pkg/apis/trial/v1alpha1/echofield_types.go
+++ b/kubebuilder-test-controller/pkg/apis/trial/v1alpha1/echofield_types.go
@@ -27,12 +27,14 @@ import (
 type EchoFieldSpec struct {
 	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
 	// Important: Run "make" to regenerate code after modifying this file
+	Field string `json:"field"`
 }
 
 // EchoFieldStatus defines the observed state of EchoField
 type EchoFieldStatus struct {
 	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
 	// Important: Run "make" to regenerate code after modifying this file
+	Field string `json:"field,omitempty"`
 }
 
 // +genclient
diff --git a/kubebuilder-test-controller/pkg/controller/echofield/echofield_controller.go b/kubebuilder-test-controller/pkg/controller/echofield/echofield_controller.go
index 3f06425..7185bfe 100644
--- a/kubebuilder-test-controller/pkg/controller/echofield/echofield_controller.go
+++ b/kubebuilder-test-controller/pkg/controller/echofield/echofield_controller.go
@@ -18,18 +18,13 @@ package echofield
 
 import (
 	"context"
-	"reflect"
 
 	trialv1alpha1 "github.com/mizzy/kubernetes-crd-playground/kubebuilder-test-controller/pkg/apis/trial/v1alpha1"
 	appsv1 "k8s.io/api/apps/v1"
-	corev1 "k8s.io/api/core/v1"
 	"k8s.io/apimachinery/pkg/api/errors"
-	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
 	"k8s.io/apimachinery/pkg/runtime"
-	"k8s.io/apimachinery/pkg/types"
 	"sigs.k8s.io/controller-runtime/pkg/client"
 	"sigs.k8s.io/controller-runtime/pkg/controller"
-	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
 	"sigs.k8s.io/controller-runtime/pkg/handler"
 	"sigs.k8s.io/controller-runtime/pkg/manager"
 	"sigs.k8s.io/controller-runtime/pkg/reconcile"
@@ -113,55 +108,15 @@ func (r *ReconcileEchoField) Reconcile(request reconcile.Request) (reconcile.Res
 		return reconcile.Result{}, err
 	}
 
-	// TODO(user): Change this to be the object type created by your controller
-	// Define the desired Deployment object
-	deploy := &appsv1.Deployment{
-		ObjectMeta: metav1.ObjectMeta{
-			Name:      instance.Name + "-deployment",
-			Namespace: instance.Namespace,
-		},
-		Spec: appsv1.DeploymentSpec{
-			Selector: &metav1.LabelSelector{
-				MatchLabels: map[string]string{"deployment": instance.Name + "-deployment"},
-			},
-			Template: corev1.PodTemplateSpec{
-				ObjectMeta: metav1.ObjectMeta{Labels: map[string]string{"deployment": instance.Name + "-deployment"}},
-				Spec: corev1.PodSpec{
-					Containers: []corev1.Container{
-						{
-							Name:  "nginx",
-							Image: "nginx",
-						},
-					},
-				},
-			},
-		},
-	}
-	if err := controllerutil.SetControllerReference(instance, deploy, r.scheme); err != nil {
-		return reconcile.Result{}, err
-	}
-
-	// TODO(user): Change this for the object type created by your controller
-	// Check if the Deployment already exists
-	found := &appsv1.Deployment{}
-	err = r.Get(context.TODO(), types.NamespacedName{Name: deploy.Name, Namespace: deploy.Namespace}, found)
-	if err != nil && errors.IsNotFound(err) {
-		log.Info("Creating Deployment", "namespace", deploy.Namespace, "name", deploy.Name)
-		err = r.Create(context.TODO(), deploy)
-		return reconcile.Result{}, err
-	} else if err != nil {
-		return reconcile.Result{}, err
-	}
-
-	// TODO(user): Change this for the object type created by your controller
-	// Update the found object and write the result back if there are any changes
-	if !reflect.DeepEqual(deploy.Spec, found.Spec) {
-		found.Spec = deploy.Spec
-		log.Info("Updating Deployment", "namespace", deploy.Namespace, "name", deploy.Name)
-		err = r.Update(context.TODO(), found)
+	desire := instance.DeepCopy()
+	if desire.Status.Field != instance.Spec.Field {
+		desire.Status.Field = instance.Spec.Field
+		log.Info("Updating Echofield %s%s\n", instance.Namespace, instance.Name)
+		err = r.Update(context.TODO(), desire)
 		if err != nil {
 			return reconcile.Result{}, err
 		}
 	}
+
 	return reconcile.Result{}, nil
 }

```

```
make
make install
make run
```

`make` may show errors, but you can ignore them.


`echofield.yaml`:

```yaml
apiVersion: trial.cstoku.io/v1alpha1
kind: EchoField
metadata:
  name: echofield-sample
spec:
  field: "hello!!"
```

```
$ kubectl apply -f echofield.yaml
echofield.trial.cstoku.io/echofield-sample created

$ kubectl get -f echofield.yaml -o yaml

apiVersion: trial.mizzy.org/v1alpha1
kind: EchoField
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"trial.mizzy.org/v1alpha1","kind":"EchoField","metadata":{"annotations":{},"name":"echofield-sample","namespace":"crd"},"spec":{"field":"hello!!"}}
  creationTimestamp: "2019-03-13T09:21:35Z"
  generation: 1
  name: echofield-sample
  namespace: crd
  resourceVersion: "319506"
  selfLink: /apis/trial.mizzy.org/v1alpha1/namespaces/crd/echofields/echofield-sample
  uid: 65fa7cac-4571-11e9-9ad2-080027f42b90
spec:
  field: hello!!
status:
  field: hello!!
```


