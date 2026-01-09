1) Configure the Identity Provider for the Openshift.
●​ Create an htpasswd Identity Provider with the name: htpass-ex280
●​ Create the secret for Identity provider users: htpass-idp-ex280
●​ Create the user accoun​ with password jobs123
●​ Create the user account wozniak with password wozniak123
●​ Create the user account collins with password collins123
●​ Create the user account adlerin with password adlerin123
●​ Create the user account armstrong with password armstrong123
=============================================================================

Step-by-Step Guide
1. Create the htpasswd File and User Accounts

First, we need to create the htpasswd file that contains all the users and their encrypted passwords.

a) Install the httpd-tools package (if not already installed): This package provides the htpasswd command.

# On RHEL/CentOS/Fedora
sudo dnf install httpd-tools

# On Ubuntu/Debian
sudo apt-get install apache2-utils
copy

b) Create the htpasswd file and add the first user: We'll create a file named htpasswd-ex280 and add the user jobs.

htpasswd -c -B -b htpasswd-ex280 jobs jobs123
copy

    -c: Create a new file.
    -B: Use the bcrypt encryption algorithm (highly recommended for security).
    -b: Read the password from the command line (for scripts). In an interactive setting, you can omit this and you will be prompted.

c) Add the remaining users to the same file: Notice we do not use the -c flag here, as it would overwrite the existing file.
``
htpasswd -B -b htpasswd-ex280 wozniak wozniak123
htpasswd -B -b htpasswd-ex280 collins collins123
htpasswd -B -b htpasswd-ex280 adlerin adlerin123
htpasswd -B -b htpasswd-ex280 armstrong armstrong123
copy
``
d) Verify the file contents:

cat htpasswd-ex280
copy

The output should look similar to this (the encrypted passwords will be different):

jobs:$2y$05$V4zX6S6P6Qm6U6Q6Q6Q6QO6Q6Q6Q6Q6Q6Q6Q6Q6Q6Q6Q6Q6Q6Q
wozniak:$2y$05$X5yY7T7Q7Rm7V7V7V7V7VP7V7V7V7V7V7V7V7V7V7V7V7V7V7V
collins:$2y$05$W6zX8U8R8Sn8X8X8X8X8XS8X8X8X8X8X8X8X8X8X8X8X8X8X8X
adlerin:$2y$05$Z7aY9V9T9To9Z9Z9Z9Z9ZT9Z9Z9Z9Z9Z9Z9Z9Z9Z9Z9Z9Z9Z9Z
armstrong:$2y$05$A8bZ0W0U0Up0A0A0A0A0AU0A0A0A0A0A0A0A0A0A0A0A0A0A0A

2. Create the OpenShift Secret from the htpasswd File

Now, we create the Secret named htpass-idp-ex280 in the openshift-config namespace, which holds the cluster-wide configuration.

oc create secret generic htpass-idp-ex280 \
  --from-file=htpasswd=./htpasswd-ex280 \
  -n openshift-config
copy

Verify the secret was created:

oc get secret htpass-idp-ex280 -n openshift-config
copy

3. Configure the OAuth Custom Resource to Use the htpasswd Identity Provider

The OAuth cluster resource manages identity providers. We need to patch it to add our new htpasswd configuration.

a) Apply the configuration patch: This command adds the htpass-ex280 identity provider to the OAuth configuration without affecting any existing providers.

oc patch oauth cluster \
  --type='merge' \
  --patch '{"spec":{"identityProviders":[{"name":"htpass-ex280","mappingMethod":"claim","type":"HTPasswd","htpasswd":{"fileData":{"name":"htpass-idp-ex280"}}}]}}'
copy

    name: htpass-ex280: The name of the Identity Provider as requested.
    type: HTPasswd: Specifies the identity provider type.
    fileData.name: htpass-idp-ex280: The name of the secret we created in the previous step.
    mappingMethod: claim: Determines how users are mapped to OpenShift user identities. claim is the standard method.

b) (Alternative) Use oc edit for more control: You can also directly edit the OAuth resource. This is often safer if you are managing multiple providers.

oc edit oauth cluster
copy

Then, inside the editor, add the identityProvider section to the spec: block. It should look like this when done:

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: htpass-ex280
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-idp-ex280
  # ... other existing identity providers may be here ...
copy

Save and exit the editor. The API server will apply the changes.
4. Wait for the Configuration to Rollout

The OAuth server pods (and other related components) need to restart to pick up the new configuration. This happens automatically but takes a minute or two.

You can watch the rollout progress:

oc get pods -n openshift-authentication
copy

Wait until all the pods are in the Running state again.
5. Test the Login

Now you can log out of the OpenShift web console or CLI and log back in using one of the new user accounts.

a) Using the CLI:

oc login -u jobs
# You will be prompted for the password: enter 'jobs123'
copy

The first time a user logs in, a User and Identity resource will be automatically created for them.

b) Using the Web Console:

    Go to the console URL.
    Select the authentication method htpass-ex280.
    Enter the username (e.g., jobs) and password (jobs123).
    Click Log In.

6. (Optional) Verify User Creation

After successfully logging in with a user (e.g., jobs), you can check that the user resource was created.

# First, switch back to an admin context if you logged in as 'jobs'
oc login -u kubeadmin # (or your admin user)

# Now, list the users
oc get users
copy

You should see all five users (jobs, wozniak, collins, adlerin, armstrong) listed.
Summary of Commands Executed

Here is the complete sequence of commands used to achieve the goal:

# 1. Create the htpasswd file and users
htpasswd -c -B -b htpasswd-ex280 jobs jobs123
htpasswd -B -b htpasswd-ex280 wozniak wozniak123
htpasswd -B -b htpasswd-ex280 collins collins123
htpasswd -B -b htpasswd-ex280 adlerin adlerin123
htpasswd -B -b htpasswd-ex280 armstrong armstrong123

# 2. Create the OpenShift Secret
oc create secret generic htpass-idp-ex280 --from-file=htpasswd=./htpasswd-ex280 -n openshift-config

# 3. Configure the OAuth resource to use the new IdP
oc patch oauth cluster --type='merge' --patch '{"spec":{"identityProviders":[{"name":"htpass-ex280","mappingMethod":"claim","type":"HTPasswd","htpasswd":{"fileData":{"name":"htpass-idp-ex280"}}}]}}'

# 4. Wait for pods to restart, then test login
oc login -u jobs
# Enter password: jobs123
