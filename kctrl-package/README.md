
# INSTALL

## configure folders
```shell
mkdir repo # package repo lives here
mkdir petclinic # petclinic package goes here
cd petclinic
mkdir config # k8s resources go here
```

## populate k8s resources into petclinic/config

## create package files
```
kctrl package init

Welcome! Before we start, do install the latest Carvel suite of tools,
specifically ytt, imgpkg, vendir and kbld.

Basic Information
A package reference name must be at least three '.' separated segments,e.g.
samplepackage.corp.com
> Enter the package reference name (samplepackage.corp.com): kctrl-petclinic.gtt.tanzu.vmware.com

Content
Please provide the location from where your Kubernetes manifests or Helm chart
can be fetched. This will be bundled as a part of the package.
1: Local Directory
2: Github Release
3: Helm Chart from Helm Repository
4: Git Repository
5: Helm Chart from Git Repository
> Enter source (1): 1

We need to include files/ directories which contain Kubernetes manifests.
Multiple values can be included using a comma separator.
> Enter the paths which contain Kubernetes manifests (): config

Output
Successfully updated package-build.yml
Successfully updated package-resources.yml

Next steps
Created files can be consumed in following ways:
1. `package release` command to release the package.
2. `package release --repo-output repo` to release the package and add it to the
package repository directory.

Succeeded
```

```shell
l
total 16
drwxr-xr-x  4 arobert  staff   128B Dec  1 12:47 .
drwxr-xr-x  3 arobert  staff    96B Dec  1 12:47 ..
-rwxr-xr-x  1 arobert  staff   385B Dec  1 12:47 package-build.yml
-rwxr-xr-x  1 arobert  staff   1.1K Dec  1 12:47 package-resources.yml
```

list initial files

package-build.yml
```shell
apiVersion: kctrl.carvel.dev/v1alpha1
kind: PackageBuild
metadata:
  creationTimestamp: null
  name: kctrl-petclinic.gtt.tanzu.vmware.com
spec:
  template:
    spec:
      app:
        spec:
          deploy:
          - kapp: {}
          template:
          - ytt:
              paths:
              - config
          - kbld: {}
      export:
      - includePaths:
        - config

```
package-resources.yml
```shell
apiVersion: data.packaging.carvel.dev/v1alpha1
kind: Package
metadata:
  creationTimestamp: null
  name: kctrl-petclinic.gtt.tanzu.vmware.com.0.0.0
spec:
  refName: kctrl-petclinic.gtt.tanzu.vmware.com
  releasedAt: null
  template:
    spec:
      deploy:
      - kapp: {}
      fetch:
      - git: {}
      template:
      - ytt:
          paths:
          - config
      - kbld: {}
  valuesSchema:
    openAPIv3: null
  version: 0.0.0

---
apiVersion: data.packaging.carvel.dev/v1alpha1
kind: PackageMetadata
metadata:
  creationTimestamp: null
  name: kctrl-petclinic.gtt.tanzu.vmware.com
spec:
  displayName: kctrl-petclinic
  longDescription: kctrl-petclinic.gtt.tanzu.vmware.com
  shortDescription: kctrl-petclinic.gtt.tanzu.vmware.com

---
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageInstall
metadata:
  annotations:
    kctrl.carvel.dev/local-fetch-0: .
  creationTimestamp: null
  name: kctrl-petclinic
spec:
  packageRef:
    refName: kctrl-petclinic.gtt.tanzu.vmware.com
    versionSelection:
      constraints: 0.0.0
  serviceAccountName: kctrl-petclinic-sa
status:
  conditions: null
  friendlyDescription: ""
  observedGeneration: 0
```

## update generated resources

update `repo/packages/package_name/metadata.yml` file (descriptions, etc.)

## release our first package

```shell
kctrl package release --version 0.0.1 --tag 0.0.1 --repo-output ../repo     

Prerequisites
1. Host is authorized to push images to a registry (can be set up by running
`docker login`)
2. `package init` ran successfully.

The bundle created needs to be pushed to an OCI registry. (format:
<REGISTRY_URL/REPOSITORY_NAME>) e.g. index.docker.io/k8slt/sample-bundle
> Enter the registry URL (): projects.registry.vmware.com/gtt/packages/kctrl-petclinic

kbld builds images when necessary and ensures that all image references are
resolved to an immutable reference
Building images and resolving references
	    | $ ytt -f /var/folders/17/r5s_f4zn5d32ff7zh21zs9g40000gp/T/kapp-controller-fetch-template-deploy3637884382/0/config
	    | $ kbld -f - --imgpkg-lock-output=.imgpkg/images.yml

An imgpkg bundle consists of all required manifests bundled into an OCI image.
This image is pushed to a registry and consumed by the package.
Pushing imgpkg bundle
	    | $ imgpkg push -b projects.registry.vmware.com/gtt/packages/kctrl-petclinic:0.0.1 -f ./bundle-projects.registry.vmware.com-gtt-packages-kctrl-petclinic:0.0.1-3807614423 --tty=true
	    | dir: .
	    | dir: .imgpkg
	    | file: .imgpkg/images.yml
	    | dir: config
	    | file: config/config.yml
	    | file: config/values.yml
	    | Pushed 'projects.registry.vmware.com/gtt/packages/kctrl-petclinic@sha256:bf0e1f5064c4e7eaf42b13006829d71bc19e8001edbd502a33a5b963778f7866'
	    | Succeeded
Artifact created: carvel-artifacts/packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
Artifact created: carvel-artifacts/packages/kctrl-petclinic.gtt.tanzu.vmware.com/package.yml
Artifact created: ../repo/packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
Artifact created: ../repo/packages/kctrl-petclinic.gtt.tanzu.vmware.com/0.0.1.yml

Next steps
1. The artifacts generated by the `--repo-output` flag can be bundled into a
PackageRepository by using the `package repository release` command.
2. Generated Package and PackageMetadata manifests can be applied to the cluster
directly.

Succeeded

```

## release the repo (publish the package for consumption)

2. release the repo
```shell
kctrl package repository release -v 0.0.1   

Prerequisites
1. `packages` directory containing Package and PackageMetadata files present in
the working directory.
2. Host is authorized to push images to a registry (can be set up using `docker
login`)

Basic Information

> Enter the package repository name (sample-repo.carvel.dev): kctrl-gtt-repo

Registry URL
The bundle created needs to be pushed to an OCI registry (format:
<REGISTRY_URL/REPOSITORY_NAME>) e.g. index.docker.io/k8slt/sample-bundle
> Enter the registry url (): projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo

kbld ensures that all image references are resolved to an immutable reference.
Lock image references using kbld
	    | $ kbld -f packages --imgpkg-lock-output .imgpkg/images.yml

An imgpkg bundle consists of all required YAML configuration bundled into an OCI
image that can be pushed to an image registry and consumed by the package.
Pushing imgpkg bundle
	    | $ imgpkg push -b projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo:0.0.1 -f ./bundle-projects.registry.vmware.com-gtt-packages-kctrl-gtt-repo:0.0.1-2349729499 --tty=true
	    | dir: .
	    | dir: .imgpkg
	    | file: .imgpkg/images.yml
	    | dir: packages
	    | dir: packages/kctrl-petclinic.gtt.tanzu.vmware.com
	    | file: packages/kctrl-petclinic.gtt.tanzu.vmware.com/0.0.1.yml
	    | file: packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
	    | Pushed 'projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo@sha256:9e6fd4989d0b2baf7d9d3598db87d42c1b973487c9ed8ffb62cdbde3588f8bfd'
	    | Succeeded
Successfully created package-repository.yml

Next steps
1. Add the package repository to the cluster by running `package repository add`
2. Alternatively, apply 'package-repository.yml' directly to your cluster.

Succeeded

```

## add repo to cluster

```shell
tanzu package repository add kctrl-gtt-repo \
    --url projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo:0.0.1 \
    --namespace tanzu-package-repo-global \
    --create-namespace

 Adding package repository 'kctrl-gtt-repo'
 Validating provided settings for the package repository
 Creating namespace 'tanzu-package-repo-global'
 Creating package repository resource
 Waiting for 'PackageRepository' reconciliation for 'kctrl-gtt-repo'
 'PackageRepository' resource install status: Reconciling
 'PackageRepository' resource install status: ReconcileSucceeded
Added package repository 'kctrl-gtt-repo' in namespace 'tanzu-package-repo-global'

```

## verify package repo install
```shell
tanzu package repository get kctrl-gtt-repo --namespace tanzu-package-repo-global

NAME:          kctrl-gtt-repo
VERSION:       29224525
REPOSITORY:    projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo
TAG:           0.0.1
STATUS:        Reconcile succeeded
REASON:        
```


## install package

### find version you want

```shell
tanzu package available list -A

  NAME                                                 DISPLAY-NAME             SHORT-DESCRIPTION                              LATEST-VERSION         NAMESPACE                  
  k8s-dashboard.gtt.tanzu.vmware.com                   k8s-dashboard            k8s-dashboard.gtt.tanzu.vmware.com             0.0.3                  tanzu-package-repo-global  
  kctrl-petclinic.gtt.tanzu.vmware.com                 kctrl-petclinic          kctrl-petclinic.gtt.tanzu.vmware.com           0.0.1                  tanzu-package-repo-global  
  petclinic.gtt.tanzu.vmware.com                       petclinic                spring petclinic                               0.0.2                  tanzu-package-repo-global  
  wavefrontproxy.gtt.tanzu.vmware.com                  wavefrontproxy           wavefrontproxy.gtt.tanzu.vmware.com            0.0.14                 tanzu-package-repo-global   
```


### generate default values file

```shell
tanzu package available get kctrl-petclinic.gtt.tanzu.vmware.com/0.0.1 \
   --namespace  tanzu-package-repo-global \
   --generate-default-values-file

NAME:                             kctrl-petclinic.gtt.tanzu.vmware.com
VERSION:                          0.0.1
RELEASED-AT:                      2022-12-01 12:58:44 -0700 MST
DISPLAY-NAME:                     kctrl-petclinic
SHORT-DESCRIPTION:                kctrl-petclinic.gtt.tanzu.vmware.com
PACKAGE-PROVIDER:                 
MINIMUM-CAPACITY-REQUIREMENTS:    
LONG-DESCRIPTION:                 kctrl-petclinic.gtt.tanzu.vmware.com
MAINTAINERS:                      []
RELEASE-NOTES:                    
LICENSE:                          []
SUPPORT:                          
CATEGORY:                         []

Created default values file at /Users/arobert/dev/repos/acme-fitness-store/kctrl-package/kctrl-petclinic-default-values.yaml
```

### edit file

### install package using values file

```shell
tanzu package install kctrl-petclinic \
    --package-name kctrl-petclinic.gtt.tanzu.vmware.com \
    --namespace kctrl-petclinic \
    --version 0.0.1 \
    --values-file kctrl-petclinic-default-values.yaml \
    --create-namespace

 Installing package 'kctrl-petclinic.gtt.tanzu.vmware.com'
 Creating namespace 'kctrl-petclinic'
 Getting package metadata for 'kctrl-petclinic.gtt.tanzu.vmware.com'
 Creating service account 'kctrl-petclinic-kctrl-petclinic-sa'
 Creating cluster admin role 'kctrl-petclinic-kctrl-petclinic-cluster-role'
 Creating cluster role binding 'kctrl-petclinic-kctrl-petclinic-cluster-rolebinding'
 Creating secret 'kctrl-petclinic-kctrl-petclinic-values'
 Creating package resource
 Waiting for 'PackageInstall' reconciliation for 'kctrl-petclinic'
 'PackageInstall' resource install status: Reconciling
 'PackageInstall' resource install status: ReconcileSucceeded

 Added installed package 'kctrl-petclinic'

```

### verify package install

```shell
tanzu package installed get kctrl-petclinic --namespace kctrl-petclinic

NAME:                    kctrl-petclinic
PACKAGE-NAME:            kctrl-petclinic.gtt.tanzu.vmware.com
PACKAGE-VERSION:         0.0.1
STATUS:                  Reconcile succeeded
CONDITIONS:              [{ReconcileSucceeded True  }]
USEFUL-ERROR-MESSAGE:    

```

# UPDATE

## update + release package

1. make changes to resources
2. release
```shell
kctrl package release --version 0.0.2 --tag 0.0.2 --repo-output ../repo  

Prerequisites
1. Host is authorized to push images to a registry (can be set up by running
`docker login`)
2. `package init` ran successfully.

The bundle created needs to be pushed to an OCI registry. (format:
<REGISTRY_URL/REPOSITORY_NAME>) e.g. index.docker.io/k8slt/sample-bundle
> Enter the registry URL (projects.registry.vmware.com/gtt/packages/kctrl-petclinic): 

kbld builds images when necessary and ensures that all image references are
resolved to an immutable reference
Building images and resolving references
	    | $ ytt -f /var/folders/17/r5s_f4zn5d32ff7zh21zs9g40000gp/T/kapp-controller-fetch-template-deploy493999397/0/config
	    | $ kbld -f - --imgpkg-lock-output=.imgpkg/images.yml

An imgpkg bundle consists of all required manifests bundled into an OCI image.
This image is pushed to a registry and consumed by the package.
Pushing imgpkg bundle
	    | $ imgpkg push -b projects.registry.vmware.com/gtt/packages/kctrl-petclinic:0.0.2 -f ./bundle-projects.registry.vmware.com-gtt-packages-kctrl-petclinic:0.0.2-938004464 --tty=true
	    | dir: .
	    | dir: .imgpkg
	    | file: .imgpkg/images.yml
	    | dir: config
	    | file: config/config.yml
	    | file: config/values.yml
	    | Pushed 'projects.registry.vmware.com/gtt/packages/kctrl-petclinic@sha256:bf0e1f5064c4e7eaf42b13006829d71bc19e8001edbd502a33a5b963778f7866'
	    | Succeeded
Artifact created: carvel-artifacts/packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
Artifact created: carvel-artifacts/packages/kctrl-petclinic.gtt.tanzu.vmware.com/package.yml
Artifact created: ../repo/packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
Artifact created: ../repo/packages/kctrl-petclinic.gtt.tanzu.vmware.com/0.0.2.yml

Next steps
1. The artifacts generated by the `--repo-output` flag can be bundled into a
PackageRepository by using the `package repository release` command.
2. Generated Package and PackageMetadata manifests can be applied to the cluster
directly.

Succeeded

```

## update + release repo

```shell
kctrl package repository release -v 0.0.2

Prerequisites
1. `packages` directory containing Package and PackageMetadata files present in
the working directory.
2. Host is authorized to push images to a registry (can be set up using `docker
login`)

Basic Information

> Enter the package repository name (kctrl-gtt-repo): 

Registry URL
The bundle created needs to be pushed to an OCI registry (format:
<REGISTRY_URL/REPOSITORY_NAME>) e.g. index.docker.io/k8slt/sample-bundle
> Enter the registry url (projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo): 

kbld ensures that all image references are resolved to an immutable reference.
Lock image references using kbld
	    | $ kbld -f packages --imgpkg-lock-output .imgpkg/images.yml

An imgpkg bundle consists of all required YAML configuration bundled into an OCI
image that can be pushed to an image registry and consumed by the package.
Pushing imgpkg bundle
	    | $ imgpkg push -b projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo:0.0.2 -f ./bundle-projects.registry.vmware.com-gtt-packages-kctrl-gtt-repo:0.0.2-2693752490 --tty=true
	    | dir: .
	    | dir: .imgpkg
	    | file: .imgpkg/images.yml
	    | dir: packages
	    | dir: packages/kctrl-petclinic.gtt.tanzu.vmware.com
	    | file: packages/kctrl-petclinic.gtt.tanzu.vmware.com/0.0.1.yml
	    | file: packages/kctrl-petclinic.gtt.tanzu.vmware.com/0.0.2.yml
	    | file: packages/kctrl-petclinic.gtt.tanzu.vmware.com/metadata.yml
	    | Pushed 'projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo@sha256:651e6b3d80be2ccd95f9794a0ca07d9b9f4bc0917b3dc15c5dcd29f278bd31a5'
	    | Succeeded
Successfully created package-repository.yml

Next steps
1. Add the package repository to the cluster by running `package repository add`
2. Alternatively, apply 'package-repository.yml' directly to your cluster.

Succeeded

```


## update cluster

### update repo
```shell
tanzu package repository add kctrl-gtt-repo \
    --url projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo:0.0.2 \
    --namespace tanzu-package-repo-global \
    --create-namespace

 Adding package repository 'kctrl-gtt-repo'
 Updating package repository 'kctrl-gtt-repo'
 Getting package repository 'kctrl-gtt-repo'
 Validating provided settings for the package repository
 Updating package repository resource
 Waiting for 'PackageRepository' reconciliation for 'kctrl-gtt-repo'
 'PackageRepository' resource install status: Reconciling


Updated package repository 'kctrl-gtt-repo' in namespace 'tanzu-package-repo-global'
```

### verify package repo updated

```shell
tanzu package repository get kctrl-gtt-repo --namespace tanzu-package-repo-global

NAME:          kctrl-gtt-repo
VERSION:       29248399
REPOSITORY:    projects.registry.vmware.com/gtt/packages/kctrl-gtt-repo
TAG:           0.0.2
STATUS:        Reconcile succeeded
REASON:        
```

```shell
tanzu package available list -A

  NAME                                                 DISPLAY-NAME                 SHORT-DESCRIPTION                                  LATEST-VERSION         NAMESPACE                  
  k8s-dashboard.gtt.tanzu.vmware.com                   k8s-dashboard                k8s-dashboard.gtt.tanzu.vmware.com                 0.0.3                  tanzu-package-repo-global  
  kctrl-petclinic.gtt.tanzu.vmware.com                 kctrl-petclinic              spring pet clinic packaged using kctrl             0.0.2                  tanzu-package-repo-global    
```


### update package

```shell
tanzu package installed update kctrl-petclinic \
    --package-name kctrl-petclinic.gtt.tanzu.vmware.com \
    --namespace kctrl-petclinic \
    --version 0.0.2 \
    --values-file kctrl-petclinic-default-values.yaml

 Updating installed package 'kctrl-petclinic'
 Getting package install for 'kctrl-petclinic'
 Getting package metadata for 'kctrl-petclinic.gtt.tanzu.vmware.com'
 Updating secret 'kctrl-petclinic-kctrl-petclinic-values'
 Updating package install for 'kctrl-petclinic'
 Waiting for 'PackageInstall' reconciliation for 'kctrl-petclinic'
 'PackageInstall' resource install status: ReconcileSucceeded
Updated installed package 'kctrl-petclinic' in namespace 'kctrl-petclinic'

```

### verify package update successful

```shell
tanzu package installed get kctrl-petclinic --namespace kctrl-petclinic

NAME:                    kctrl-petclinic
PACKAGE-NAME:            kctrl-petclinic.gtt.tanzu.vmware.com
PACKAGE-VERSION:         0.0.2
STATUS:                  Reconcile succeeded
CONDITIONS:              [{ReconcileSucceeded True  }]
USEFUL-ERROR-MESSAGE:    

```