# openshift-keycloak



## Getting started

Add the resources to openshift to install the RH-SSO Operator into a specific namespace 'rh-sso'. 
Note the resources are configured with the namespace name 'rh-sso'.  If you wish to install the operator
into a different namespace, you will need to edit the .yaml files appropriately.

**The following will require cluster admin priveledges. **

```

oc new-project rh-sso
oc apply -f templates/operatorgroup.yaml
oc apply -f templates/operator.yaml

```

The above command will add the necessary resources (Subscription and OperatorGroup) to install the RH-SSO operator into the 'rh-sso' namespace. 


Create ClusterRole 'keycloak-project-admin' that gives a (project admin) user necessary privileges to install Keycloak (and associated resources) via the RH-SSO Operator: 

```

$ oc apply -f templates/cluster-role-keycloak-project-admin.yaml 
$ oc adm policy add-role-to-user keycloak-project-admin <user name> -n rh-sso

```
**Subsequent commands must be run as either 'cluster-admin', or as a 'project-admin' user with apprpriate cluster role 'keycloak-project-admin'
applied to the user.** 

If the secret doesn't already exist in the namespace, a secret holding the user/password for the Keycloak admin console gets created 
automatically with a random password when you apply the Keycloak CR. The default name of this secret is  
'credential-${keycloak name}' where 'keycloak name' is the value of 'metadata.name' in the keycloak CR yaml file. 
You also have the option to create the secret beforehand and it will use this secret to set the admin user/password 
instead of a random value. If you wish to specify the password for Keycloak Admin consle you can run the following command 
to create the secret prior to installing the keycloak CR. Note you should use appropriate value for ${keycloak name} and ${admin password}.

```
$ oc create secret generic credentials-${keycloak name} --from-literal ADMIN_USERNAME=admin --from-literal ADMIN_PASSWORD=${admin password}

```

Appply the CR to create keycloak instance and associated resources.

```
$ oc apply -f templates/keycloak.yaml 
```

If you created the secret containing admin user/pass beforehand you can proceed to login to the admin console.  Get the URL by 
runnint 'oc get routes' command.  If you did not create a custom user/password, the following commands can be used to get password where keycloak instance 'mykeycloak' user/pass credentials kept (alternatively you can find the values in the 'secrets' in the Openshift console.) 

```

$ oc get keycloak mykeycloak --output="jsonpath={.status.credentialSecret}"
$ oc get secret <secret name returned above> -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'

```

Check status of pod 

```

$ oc get keycloak 
$ oc describe keycloak mykeycloak

```

To delete - this deletes all CRs in the yaml, including secrets.

```

$ oc delete -f templates/keycloak.yaml

```
To delete the operator 

```
$ oc delete 
```