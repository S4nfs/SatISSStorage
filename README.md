## Testing

1. Create a `deployment.yaml` file to deploy Kubernetes resources for the storage operator or driver in the development environment. Review the following example `deployment.yaml` to deploy the `local-volume` operator.
   ```yaml
   apiVersion: v1
   kind: List
   metadata:
     name: local-storage
     namespace: kube-system
     annotations:
       version: local-storage-45
   items:
     - apiVersion: v1 [1]
       kind: Namespace
       metadata:
         name: local-storage
     - apiVersion: operators.coreos.com/v1alpha2 [2]
       kind: OperatorGroup
       metadata:
         name: local-operator-group
         namespace: local-storage
       spec:
         targetNamespaces:
           - local-storage
     - apiVersion: operators.coreos.com/v1alpha1 [3]
       kind: Subscription
       metadata:
         name: local-storage-operator
         namespace: local-storage
       spec:
         channel: '4.5'
         installPlanApproval: Automatic
         name: local-storage-operator
         source: redhat-operators
         sourceNamespace: openshift-marketplace
     - apiVersion: local.storage.openshift.io/v1 [4]
       kind: LocalVolume
       metadata:
         name: local-disk
         namespace: local-storage
       spec:
         nodeSelector:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: storage
                   operator: In
                   values:
                     - 'localvol'
         storageClassDevices:
           - storageClassName: 'localblock-sc'
             volumeMode: Block
             devicePaths:
               - /dev/xvde
   ```
1. Test your deployment.

   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Create Configuration`
   1. From `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Add version and upload the deployment yaml`
   1. From the `IBM Cloud Console` -> `Satellite` -> `Configurations` -> `Select the Configuration` -> `Create Subscription`
   1. Verify that the resources are deployment to the cluster.
   1. Create a PVC and run the sniff test using the IBM Provides test bucket to make sure deployment is working as expected.

1. After you have verified your template in the development environment, update the issue. Include the following information in your issue and ensure that you remove any secrets from your files.
   1. Your `deployment.yaml` file.
   1. An example`pvc.yaml` file.
   1. An example `app.yaml`.
   1. The output of the `oc get sc` command that displays the storage classes your deployment creates.
   1. The output of the `oc get -n <namespace> pvc` command that displays the PVCs your deployment creates.
   1. The output of the `oc get -n <namespace> pods` command that displays the pods your deployment creates.
