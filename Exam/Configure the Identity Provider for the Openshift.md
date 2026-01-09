📘 Configure HTPasswd Identity Provider in OpenShift

This guide explains how to configure an HTPasswd Identity Provider in OpenShift, create users, and link them to the cluster.

✅ Prerequisites

OpenShift cluster access

oc CLI installed and logged in as cluster-admin

Linux system with htpasswd utility

sudo dnf install httpd-tools -y

🧾 Configuration Details
Item	Value
Identity Provider Name	htpass-ex280
Secret Name	htpass-idp-ex280
Users	jobs, wozniak, collins, adlerin, armstrong
🔹 Step 1: Create htpasswd File

Create a working directory:

mkdir ~/htpasswd-idp
cd ~/htpasswd-idp

Create users with passwords:

htpasswd -c -B -b htpasswd jobs jobs123
htpasswd -B -b htpasswd wozniak wozniak123
htpasswd -B -b htpasswd collins collins123
htpasswd -B -b htpasswd adlerin adlerin123
htpasswd -B -b htpasswd armstrong armstrong123


👉 -c is used only once (creates file)
👉 -B enables bcrypt encryption (recommended)

Verify file:

cat htpasswd

🔹 Step 2: Create Secret in OpenShift

Create the secret in openshift-config namespace:

oc create secret generic htpass-idp-ex280 \
  --from-file=htpasswd=htpasswd \
  -n openshift-config


Verify:

oc get secret htpass-idp-ex280 -n openshift-config

🔹 Step 3: Configure Identity Provider

Edit OAuth configuration:

oc edit oauth cluster

Add the following under spec.identityProviders:

spec:
  identityProviders:
  - name: htpass-ex280
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-idp-ex280


Save and exit.

⏳ OpenShift will automatically roll out OAuth pods.

🔹 Step 4: Verify OAuth Pods
oc get pods -n openshift-authentication


All pods should be in Running state.

🔹 Step 5: Test User Login

Try logging in using CLI:

oc login -u jobs -p jobs123


Repeat for other users.

🔹 Step 6: (Optional) Assign Roles

Example: give jobs admin access to a project:

oc adm policy add-role-to-user admin jobs -n my-project


Or cluster admin (⚠️ careful):

oc adm policy add-cluster-role-to-user cluster-admin jobs
