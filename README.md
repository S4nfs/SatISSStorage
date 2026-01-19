````markdown
## Testing

This section describes how to deploy, validate, and document a storage operator or driver in a development Kubernetes environment before promoting it further.

---

### 1. Create the Deployment Manifest

Create a `deployment.yaml` file that defines all the Kubernetes resources required to deploy the storage operator or driver in the **development environment**.

As an example, the following `deployment.yaml` deploys the **Local Volume Operator**. Review the file carefully and adjust values such as namespaces, versions, and device paths based on your environment.

```yaml
apiVersion: v1
kind: List
metadata:
  name: local-storage
  namespace: kube-system
  annotations:
    version: local-storage-45
items:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: local-storage

  - apiVersion: operators.coreos.com/v1alpha2
    kind: OperatorGroup
    metadata:
      name: local-operator-group
      namespace: local-storage
    spec:
      targetNamespaces:
        - local-storage

  - apiVersion: operators.coreos.com/v1alpha1
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

  - apiVersion: local.storage.openshift.io/v1
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
````

---

### 2. Deploy and Test the Configuration in IBM Cloud Satellite

After creating the deployment manifest, validate it using **IBM Cloud Satellite**.

1. In the **IBM Cloud Console**, navigate to
   **Satellite → Configurations → Create Configuration**.

2. Select the newly created configuration and:

   * Add a new version
   * Upload the `deployment.yaml` file

3. From the same configuration page:

   * Create a **Subscription** to apply the configuration to the target cluster

4. Verify that all Kubernetes resources are successfully deployed to the cluster.

5. Create a **PersistentVolumeClaim (PVC)** using the storage class created by the deployment.

6. Run a basic **sniff test** using an IBM-provided test bucket to confirm that:

   * The PVC binds successfully
   * The storage backend functions correctly
   * Pods can read and write data as expected

---

### 3. Update the Issue After Validation

Once the deployment has been successfully tested in the development environment, update the corresponding issue with the required artifacts and outputs.

> **Important:** Remove all secrets, credentials, and sensitive information before sharing files or command outputs.

Include the following in the issue:

1. The final `deployment.yaml` file.
2. An example `pvc.yaml` file.
3. An example `app.yaml` file.
4. Output of the following command showing the created storage classes:

   ```
   oc get sc
   ```
5. Output of the following command showing the PVCs in the target namespace:

   ```
   oc get -n <namespace> pvc
   ```
6. Output of the following command showing the pods created by the deployment:

   ```
   oc get -n <namespace> pods
   ```

