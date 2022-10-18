# openshift-keycloak



## Getting started

Add resources to openshift to install the RH-SSO Operator into a specific namespace 'rh-sso' 


```

oc new-project rh-sso
oc apply -f templates/operatorgroup.yaml
oc apply -f templates/operator.yaml

```

The above command will add the necessary resources (Subscription and OperatorGroup) to install the RH-SSO operator into the namespace. 

Create ClusterRole 'keycloak-project-admin' that gives a (project admin) user necessary privileges to install Keycloak (and associated resources) via the RH-SSO Operator: 

```

$ oc apply -f templates/cluster-role-keycloak-project-admin.yaml 
$ oc adm policy add-role-to-user keycloak-project-admin <user name> -n rh-sso

```

If it doesn't already exist in the namespace, a secret 'credential-${keycloak name}' gets created automatically when you apply the Keycloak CR. However the password for the admin account gets created with random admin password.  
You have the option to create the secret beforehand and it will use this secret to set the admin user/password .

```

$ oc apply -f templates/user-credentials.yaml 

```

Appply the CR to create keycloak instance and associated resources.

```
$ oc apply -f templates/keycloak.yaml 
```

Had you not created the secret containing admin user/pass beforehand the following commands could be used to get password where keycloak instance 'mykeycloak' user/pass credentials kept 

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
