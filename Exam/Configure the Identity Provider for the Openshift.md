[osuser@dcepayprodbastion ~]$ cat oauth.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.openshift.io/v1","kind":"OAuth","metadata":{"annotations":{},"name":"cluster"},"spec":{"identityProviders":[{"ldap":{"attributes":{"email":["mail"],"id":["cn"],"name":["cn"],"preferredUsername":["sAMAccountName"]},"bindDN":"cn=cmpicpdevadmin,ou=GTCMP Users,ou=GTCMP,ou=GITC Belapur,dc=corp,dc=ad,dc=sbi","bindPassword":{"name":"ldap-secret"},"ca":{"name":"ca-config-map"},"insecure":false,"url":"ldaps://corp.ad.sbi/dc=corp,dc=ad,dc=sbi?sAMAccountName"},"mappingMethod":"claim","name":"ldapidp","type":"LDAP"},{"htpasswd":{"fileData":{"name":"htpass-secret"}},"mappingMethod":"claim","name":"my_htpasswd","type":"HTPasswd"}]}}
    release.openshift.io/create-only: "true"
  creationTimestamp: "2025-07-31T09:23:17Z"
  generation: 4
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: 984ea03a-2cd3-4cff-a36f-05d41e463b3d
  resourceVersion: "3599473"
  uid: 075a25f9-a346-4766-a933-11240c21cd5c
spec:
  identityProviders:
  - ldap:
      attributes:
        email:
        - mail
        id:
        - cn
        name:
        - cn
        preferredUsername:
        - sAMAccountName
      bindDN: cn=cmpicpdevadmin,ou=GTCMP Users,ou=GTCMP,ou=GITC Belapur,dc=corp,dc=ad,dc=sbi
      bindPassword:
        name: ldap-secret
      ca:
        name: ca-config-map
      insecure: false
      url: ldaps://corp.ad.sbi/dc=corp,dc=ad,dc=sbi?sAMAccountName
    mappingMethod: claim
    name: ldapidp
    type: LDAP
  - htpasswd:
      fileData:
        name: htpass-secret
    mappingMethod: claim
    name: my_htpasswd
    type: HTPasswd
