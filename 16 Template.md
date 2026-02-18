# Install / Manage Operators
### Question: Create an project template with limitrange with container
- minimum memory is 5Mi, max is 1Gi. `defaultrequest` 254 Mi `defaultlimit` is 512 Mi .
- make sure this template available as default request `new-project` template for users.
---
```
oc adm --help
```
```
oc adm create-bootstrap-project-template -o yaml > /tmp/bootstrap.yaml
```
### Open the web console
```
oc whoami --show-console
```

### Go to "Administrator -> LimitRange", then copy the yaml file and then paste in one file.
```
vi /tmp/limitrange.yaml
```

### Modify the values as per the question.
### - minimum memory is 5Mi, max is 1Gi. `defaultrequest` 254 Mi `defaultlimit` is 512 Mi .
![image](https://github.com/user-attachments/assets/da703c9e-20c1-4805-993b-ae4c12888d3f)

### Copy the values from the above file "/tmp/limitrange.yaml" and paste it on file "/tmp/bootstrap.yaml".
```
vi /tmp/bootstrap.yaml
```
![image](https://github.com/user-attachments/assets/36f05d2d-afd8-4893-a747-5be2f68de60f)

### Create the bootstrap template.
```
oc apply -f /tmp/bootstrap.yaml -n openshift-config
```

### We can check the template.
```
oc get template -n openshift-config
```
### You must see like this output.
```
NAME               DESCRIPTION    PARAMETERS    OBJECTS
project-request                   5 (5 blank)    3
```
### - make sure this template available as default request `new-project` template for users.

```
oc get project.config
```
### You must see like this output.
![image](https://github.com/user-attachments/assets/52509e7c-8897-4b7a-aa17-3415be4b46b0)

```
oc edit project.config cluster
```

```
apiVersion: config.openshift.io/v1
kind: Project
metadata:
  annotations :
    include. release. openshift. io/ibm-cloud-managed: "true"
    include. release. openshift. io/self-managed-high-availability: "true"
    include.release. openshift. io/single-node-developer: "true"
    release. openshift. io/create-only: "true"
    creationTimestamp: "2024-01-23T12:00:53Z"
  generation: 1
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: c1595d31-17e8-4a05-9002-d7399f1ed9c3
    resourceVersion: "1633"
    uid: 1c658959-11a4-4ec5-8532-e6e5dc9a8de7
spec: 
  projectRequestTemplate:          ## Added line
    name: project-request          ## Added line

```

### Check the pods in the namespace "openshift-apiserver". 
```
oc get pods -n openshift-apiserver -w
```
### Once the new pod/s started running then, create a new project and our limitrange should be there. 
```
oc new-project anish
```
#### Check the limitrange
```
oc get limitranges 
```
To make every new project automatically include a LimitRange with your specified container memory constraints in OpenShift, you must:
Create a project template
Add a LimitRange object to it
Configure OpenShift to use this template as the default
✅ LimitRange Requirements
You specified:
Setting
Value
Minimum memory
5Mi
Maximum memory
1Gi
Default request
254Mi
Default limit
512Mi
✅ Project Template YAML
Create a file:
Copy code
project-template.yaml
With this content:
Copy code
Yaml
'''
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: custom-project-template
objects:

- apiVersion: v1
  kind: LimitRange
  metadata:
    name: container-memory-limits
  spec:
    limits:
    - type: Container
      min:
        memory: 5Mi
      max:
        memory: 1Gi
      defaultRequest:
        memory: 254Mi
      default:
        memory: 512Mi

parameters:
- name: PROJECT_NAME
  required: true

- name: PROJECT_DISPLAYNAME
  required: false

- name: PROJECT_DESCRIPTION
  required: false
  '''
✅ Load Template into OpenShift
Run:
Copy code
Bash
'''
oc create -f project-template.yaml -n openshift-config
'''
✅ Configure OpenShift to Use This Template
Edit the project configuration:
Copy code
Bash
'''
oc edit project.config.openshift.io/cluster
'''
Modify / add:
Copy code
Yaml
spec:
  projectRequestTemplate:
    name: custom-project-template
Example full section:
Copy code
Yaml
'''
spec:
  projectRequestTemplate:
    name: custom-project-template
'''
Save and exit.
✅ Result
Now whenever a user runs:
Copy code
Bash
'''
oc new-project my-test-project
'''

#### You can also create a new application.
```
oc new-app --name hello  --image registry.ocp4.example.com:8443/redhattraining/hello-world-nginx:v1.0
```
