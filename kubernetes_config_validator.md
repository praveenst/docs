# K8S Configuration Files

## Goal:
- To validate kubernetes static configuration files pre-checkin
- List any open source options and key details after tools diligence. 
- Pre-checkin or pre-release test to show Kubernetes configuration files are valid against specific versions of Kubernetes
- Test for schema compatibility to make sure the config files are still valid across releases (against master or beta releases). 

## Tools Investigated : 

### 1. kubeval
   - Opensource tool, pretty easy to install, validate against yaml files
  - Install options
    - tar file that contains a `kubeval` binary for different platforms (windows, linux, darwin) built and tagged in [Releases · garethr/kubeval · GitHub](https://github.com/garethr/kubeval/releases)
    - wget the tar file, extract the kubeval binary and copy to your local path
    - Supports homebrew installation for MacOS
    - Supports install from Chocolatey for windows users
    - Published as a docker image to run
  - Key features
    - No need to have a running cluster to validate configuration files.
    - Supports validation of  kubernetes configuration files in yaml or json schema.
    - One or more files can be validated at once, a good pre-checkin tool.
    - Can be used to validate schemas against multiple kubernetes version using `--version` option
    -  base url for schema can be passed using `--schema-location` option as a string. By default, kubeval uses those extracted schemas, published at [GitHub - garethr/kubernetes-json-schema: A set of JSON schemas for various Kubernetes versions, extracted from the OpenAPI definitions](https://github.com/garethr/kubernetes-json-schema)
    - Easy to integrate into CI by fetching the latest tag of kubeval tar file.
    - kubeval does not support CustomResourceDefinitions yet.
  - Licensing
      - Licensed under the Apache License, Version 2.0 (the "License");
      - Easy to use or redistirubute with any customization needed with some limitation that comes with Apache license. 
  - Running Kubeval
      - can be run on a command line 
```
wget https://github.com/garethr/kubeval/releases/download/0.6.0/kubeval-darwin-amd64.tar.gz
tar xf kubeval-darwin-amd64.tar.gz
cp kubeval /usr/local/bin
``` 
      - Running kubeval
        - Running against a specific version: kubeval -v 1.6.6 *.yaml
        - Running againsta default: kubeval *.yaml
```
Results running against a specific version: kubeval -v 1.6.6 *.yaml

yaml configuration files in private-kubernetes/yaml/compute

mbp-e769:compute pthangavelu$ kubeval -v 1.6.6 *.yaml
The document cm-drill.yaml contains a valid ConfigMap
The document cm-tenant.yaml contains a valid ConfigMap
The document deploy-spark-hs.yaml contains a valid Deployment
The document ns-tenant.yaml contains a valid Namespace
The document pod-spark-base.yaml contains a valid Pod
The document pod-spark-submit.yaml contains a valid Pod
The document rbac-tenant.yaml contains a valid ServiceAccount
The document rbac-tenant.yaml contains a valid ClusterRole
The document rbac-tenant.yaml contains a valid ClusterRoleBinding
The document secret-tenant-gcrimagepull.yaml contains a valid Secret
The document secret-tenant-user.yaml contains a valid Secret
The document service-spark-hs.yaml contains a valid Service
The document ss-drill.yaml contains a valid Service
The document ss-drill.yaml contains a valid Service
The document ss-drill.yaml contains a valid StatefulSet

```

### 2. copper.sh
    - This is another tool that does configuration validation of kubernetes configuration. This could be used to validate the yaml configuration files in conjunction with kubeval. This tools supports programmatic assertions through an easy to use DSL defined to write tests or any other custom validation of kubernetes configuration files. 

  - Install Options
    - Can be run from command line or docker image
    - Options to integrate it with CI/CD pipeline
    - Install using rubygems - refer copper.sh Github - [GitHub - cloud66-oss/copper: Cloud 66 Copper](https://github.com/cloud66-oss/copper)
  - Key features
    - Provides a simple DSL to validate a configuration file
    - Only supports yaml configuration files, no JSON validation.
    - Requires writing test a.k.a rule a text file
    - JSONPath syntax can be used to read configuration file
    - Using [Copper DSL](https://copper.sh/docs/copper-dsl/#rules) for configuration validation. 
    - Requrires some orientation to using Copper DSL to create tests.
   
  - Licensing
    - Copper is an open source project developed by Cloud 66. 
    - It is distributed under MPL 2.0 license

  - Running Copper.sh
    - Download instructions `sudo gem install c66-copper`
    - Create a sample ruleset and save it in a file (any text file)
    - consider assertion on config file to test `apiVersion` in `private-kubernetes/yaml/cm-drill.yaml` 
    - to test for a required condition, create rule of type `ensure
    - to run againsta single config file use `copper check --rules api_version_test.txt --file config/cm-drill.yaml`

```

Syntax:

rule NAME (warn | ensure) {
    CONDITION
}

Example: 
rule ApiV1Only ensure {
    fetch("$.apiVersion").first == "v1"
}

```

    - results shown after running the api_version_test against cm-drill.yaml

```
mbp-e769:copper.sh pthangavelu$ copper check --rules api_version_test.txt --file config/cm-drill.yaml
Validating part 0
	ApiV1Only - PASS
```

### 3. Other validation tools

  #### kubectl 
    - comes with --validate flag that you can use to test for yaml validation
    - Caveat: --validate flag does not work in conjunction with --dry-run flag

```

Test Deployment file: deployment.yaml

- This file has an intentional error in container spec (typo parts instead of ports)

apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 4
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:9.0
        parts:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 3


Running --

mbp-e769:Health Checks pthangavelu$ kubectl --context=minikube  create --validate -f deployment.yaml
error: error validating "deployment.yaml": error validating data: ValidationError(Deployment.spec.template.spec.containers[0]): unknown field "parts" in io.k8s.api.core.v1.Container; if you choose to ignore these errors, turn validation off with --validate=false


```

  #### yamllint
    - Supported on CentOS, Debian/Ubuntu16.04+ and on Mac OS 10.11+
    - Command lione based utility where you can specify a single file or an entire directory to validate
    - Entire list of supported rules are [Rules — yamllint 1.14.0 documentation](https://yamllint.readthedocs.io/en/stable/rules.html)
    - A configuration file can be used to enable/disable a default set of supported rules to curate. 





