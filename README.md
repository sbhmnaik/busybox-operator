# busybox-operator


oc login -u developer -p developer  2886795277-8443-frugo02.environments.katacoda.com
oc new-project myproject
oc get pv --as system:admin
oc new-project myproject
operator-sdk new busybox-operator --api-version=busybox.operator.com/v1alpha1 --kind=BusyBoxApp --type=ansible
cd busyboxapp-operator

########

apiVersion: apps.openshift.io/v1
apiVersion: apps.kubernetes.io/v1
apiVersion: apis.apps.openshift.io/v1
apiVersion: apps/v1

########

vi roles/busyboxapp/tasks/main.yml

---
- name: start busybox
  k8s:
    definition:
      kind: DeploymentConfig
      apiVersion: apps.openshift.io/v1
      metadata:
        name: 'busybox'
        namespace: '{{ meta.namespace }}'
      spec:
        template:
          metadata:
            labels:
              name: busybox
          spec:
            containers:
              - name: helloworld
                image: "busybox:1.28"
        replicas: "{{size}}"
        selector:
          name: busybox
        strategy:
          type: Rolling
          rollingParams:
            pre:
              failurePolicy: Abort
              execNewPod:
                containerName: helloworld
                command: ['sh', '-c', 'echo Hello from Busybox! ; sleep 600']


oc adm policy add-cluster-role-to-user cluster-admin developer -n myproject --as system:admin
oc create -f deploy/crds/busybox_v1alpha1_busyboxapp_crd.yaml -n myproject --as system:admin
 oc create -f deploy/service_account.yaml -n myproject
oc get sa
 oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:myproject:busybox-operator -n myproject --as system:admin

docker login quay.io 
operator-sdk build quay.io/<username>/busybox:v0.0.1
docker push quay.io/<username>/busybox:v0.0.1




sed -i 's|{{ REPLACE_IMAGE }}|quay.io/<username>/busybox:v0.0.1|g' deploy/operator.yaml

sed -i 's|{{ pull_policy\|default('\''Always'\'') }}|Always|g' deploy/operator.yaml



oc create -f deploy/role.yaml
oc create -f deploy/role_binding.yaml

mkdir -p /opt/ansible/roles/busyboxapp
cp -r roles/busyboxapp/ /opt/ansible/roles/busyboxapp

oc create -f deploy/operator.yaml

oc apply -f deploy/crds/busybox_v1alpha1_busyboxapp_cr.yaml

oc get deployments
oc get dc
oc logs busybox-1-hook-pre




oc delete -f deploy/service_account.yaml
oc delete -f deploy/role.yaml
oc delete -f deploy/role_binding.yaml
oc delete -f deploy/operator.yaml
oc delete -f eploy/crds/busybox_v1alpha1_busyboxapp_crd.yaml
oc delete -f deploy/crds/busybox_v1alpha1_busyboxapp_cr.yaml
----------------------------------------------------------

