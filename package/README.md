###

tanzu apps workload apply --file config/workload.yaml


### package stuff
PACKAGE_NAME=wavefront-proxy.gtt.tanzu.vmware.com
PACKAGE_FOLDER=wavefront-proxy-package-contents

PACKAGE_DISPLAY_NAME=petclinic
PACKAGE_NAME=petclinic.gtt.tanzu.vmware.com
PACKAGE_FOLDER=petclinic-package-contents
PACKAGE_VERSION=0.0.1

REPO_FOLDER=gtt-package-repo
REPO_HOST=projects.registry.vmware.com
REPO_PROJECT=gtt/packages
REPO_VERSION=0.0.20

### IMPORTANT !
### do not use hyphens inside data values names (the keys should use underscore...the default value can actually contain hyphens)


### package repo stuff

## Creating a New Package in an Existing Repo
# make folders for our package inside the package repo

    mkdir -p ${REPO_FOLDER}/.imgpkg ${REPO_FOLDER}/packages/${PACKAGE_NAME}

# Also, you'll need to create the package and add it to the repo

    mkdir -p ${PACKAGE_FOLDER} ${PACKAGE_FOLDER}/config

# Create the config.yml and values.yml

Mark up your config.yml like for example:

```yaml

#@ load("@ytt:data", "data")

---
apiVersion: v1
kind: Namespace
metadata:
name: #@ data.values.namespace
```

Here's a cool example `values.yml`:

```yaml
#@data/values-schema
---
#@schema/desc "Namespace to deploy into."
namespace: wavefront-proxy
```


# Generate the openAPIv3 schema file:

    ytt -f ${PACKAGE_FOLDER}/config/values.yml --data-values-schema-inspect -o openapi-v3 > ${PACKAGE_DISPLAY_NAME}-schema-openapi.yml


# Record the required container images with kbld

    kbld -f ${PACKAGE_FOLDER}/config/ --imgpkg-lock-output ${PACKAGE_FOLDER}/.imgpkg/images.yml

# Releasing a New Version of an existing Package

    PACKAGE_VERSION=0.0.14

# push the package


    REPO_HOST=projects.registry.vmware.com
    imgpkg push -b ${REPO_HOST}/gtt/packages/${PACKAGE_DISPLAY_NAME}:${PACKAGE_VERSION} -f ${PACKAGE_FOLDER}/

# TODO UPDATE FLOW
imgpkg push -b ${REPO_HOST}/gtt/packages/k8s-dashboard:${PACKAGE_VERSION} -f k8s-dashboard-package-contents/



At this point, the new version of the package (new image) is pushed/created to the repo HOWEVER we still need to update the package repository to include the new version of the package. Confusing? YES! At first.

# Create the package resources and schema things!

Create the package-build.yml and package-resources.yml

    kctrl package init

Now rename these files (it's fun trust me)

    mv package-build.yml wavefront-proxy-package-build.yml
    mv package-resources.yml wavefront-proxy-package-resources.yml

Now parameterize the versions in the -resources.yml file

    name: #@ "wavefront-proxy.gtt.tanzu.vmware.com." + data.values.version

Update the fetch inside the Kind:Package/spec/template/spec/fetch:
```
fetch:
- imgpkgBundle:
    image: #@ "projects.registry.vmware.com/gtt/packages/k8s-dashboard:" + data.values.version
```

as well as the kbld element in the Kind:Package/spec/template/spec/template:

```
- kbld:
    paths:
    - ".imgpkg/images.yml"
    - "-"
```

and the last section in the Kind:Package/spec:

```
valuesSchema:
  openAPIv3: #@ yaml.decode(data.values.openapi)["components"]["schemas"]["dataValues"]
version: #@ data.values.version
```

and also! kind: PackageInstall / spec / packageRef / versionSelection / constraints:

```
spec:
  packageRef:
    refName: wavefrontproxy.gtt.tanzu.vmware.com
    versionSelection:
      constraints: #@ data.values.version
```

Update the short and long description fields in the kind: PackageMetadata section:


Ex:
```
spec:
  displayName: War and Peace
  longDescription: <Russian Novel>
  shortDescription: WarAndPeace
```

In the package-build.yml file: Modify the export section to refer to the imgpkgBundle as below:

```yaml
export:
 - imgpkgBundle:
     image: projects.registry.vmware.com/gtt/packages/wavefrontproxy
     useKbldImagesLock: true
   includePaths:
    - wavefront-proxy-package-contents/config
```


# substitute the schema + version into our package definition AND create it!

    ytt -f ${PACKAGE_DISPLAY_NAME}-package-resources.yml  --data-value-file openapi=${PACKAGE_DISPLAY_NAME}-schema-openapi.yml -v version="${VERSION}" > ${REPO_FOLDER}/packages/${PACKAGE_NAME}/${VERSION}.yml

# TODO UPDATE FLOW
ytt -f k8s-dashboard-package-resources.yml  --data-value-file openapi=k8s-dashboard-schema-openapi.yml -v version="${VERSION}" > gtt-package-repo/packages/k8s-dashboard.gtt.tanzu.vmware.com/${VERSION}.yml

# Next, let's use kbld to record which package bundles are used:
# TODO ALSO UPDATE FLOW
    kbld -f gtt-package-repo/packages/ --imgpkg-lock-output gtt-package-repo/.imgpkg/images.yml

kbld -f ${REPO_FOLDER}/packages/ --imgpkg-lock-output ${REPO_FOLDER}/.imgpkg/images.yml

# lets push the repo with the updated packages inside
# TODO UPDATE FLOW

    REPO_VERSION=0.0.17

    REPO_HOST=projects.registry.vmware.com
    REPO_NAME=gtt-package-repo
    
    imgpkg push -b ${REPO_HOST}/${REPO_PROJECT}:${REPO_VERSION} -f ${REPO_FOLDER}


## Installing The Package Repo on a Cluster

### install/update package repo
### TODO UPDATE FLOW
tanzu package repository add gtt-repo \
    --url projects.registry.vmware.com/gtt/packages/gtt-package-repo:${REPO_VERSION} \
    --namespace tanzu-package-repo-global \
    --create-namespace


## verify install
tanzu package repository get gtt-repo --namespace tanzu-package-repo-global

## generate a values file
tanzu package available get wavefrontproxy.gtt.tanzu.vmware.com/0.0.1 \
   --namespace tanzu-package-repo-global \
   --generate-default-values-file


## install package
tanzu package install wavefront-proxy \
    --package-name wavefrontproxy.gtt.tanzu.vmware.com \
    --namespace wavefront-proxy \
    --version ${VERSION} \
    --values-file wavefrontproxy-default-values.yaml \
    --create-namespace

tanzu package install k8s-dashboard \
    --package-name k8s-dashboard.gtt.tanzu.vmware.com \
    --namespace k8s-dashboard \
    --version ${VERSION} \
    --values-file k8s-dashboard-default-values.yaml \
    --create-namespace


## update

tanzu package installed delete wavefront-proxy --namespace wavefront-proxy

tanzu package installed update wavefront-proxy \
    --package-name wavefrontproxy.gtt.tanzu.vmware.com \
    --namespace wavefront-proxy \
    --version ${VERSION} \
    --values-file wavefrontproxy-default-values.yaml

tanzu package installed update k8s-dashboard \
    --package-name k8s-dashboard.gtt.tanzu.vmware.com \
    --namespace k8s-dashboard \
    --version ${VERSION} \
    --values-file k8s-dashboard-default-values.yaml


## trying release stuff
VERSION=0.0.8
ytt -f wavefront-proxy-package-resources.yml  --data-value-file openapi=wavefront-proxy-schema-openapi.yml -v version="${VERSION}" > package-resources.yml
ytt -f wavefront-proxy-package-build.yml  --data-value-file openapi=wavefront-proxy-schema-openapi.yml -v version="${VERSION}" > package-build.yml

kctrl package release --version ${VERSION} --repo-output ./gtt-package-repo

projects.registry.vmware.com/gtt/packages/wavefrontproxy


k create secret

## Install the package repo
### Tanzu CLI

tanzu package repository add gtt-repo \
    --url projects.registry.vmware.com/gtt/packages/gtt-package-repo:${VERSION} \
    --namespace tanzu-package-repo-global \
    --create-namespace


# kubectl
# TODO: Explain this as a manual alternative to the above steps

apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  name: gtt-package-repo
spec:
  fetch:
    imgpkgBundle:
      image: projects.registry.vmware.com/gtt/packages/gtt-package-repo@${VERSION}
status:
  conditions: null
  friendlyDescription: ""
  observedGeneration: 0




## build repo?


apiVersion: kctrl.carvel.dev/v1alpha1
kind: PackageRepositoryBuild
metadata:
  name: gtt-package-repo
spec:
  export:
    imgpkgBundle:
    image: projects.registry.vmware.com/gtt/packages/gtt-package-repo