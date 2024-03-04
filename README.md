# GitOps Kubernetes Manifest - codefresh-gitops

## Introduction
GitOps, as a methodology, emphasizes the use of Git for managing all resources associated with an application. This extends beyond version control for source code and includes storing Kubernetes manifests within Git repositories. This document outlines the process of setting up and managing Kubernetes manifests using GitOps principles.

For further reference and detailed guides, visit: [Codefresh GitOps Deployments](https://codefresh.io/docs/docs/ci-cd-guides/gitops-deployments/
).

## Setting up your Git repositories
In GitOps, it is essential to utilize Git for managing all application resources. This entails storing not only the source code but also all Kubernetes manifests within Git repositories. Typically, this involves maintaining a separate repository for Kubernetes manifests alongside the main application repository.

## Folder Structures
- app-template: Contains an example of a template GitOps application structure. This can be customized according to your preferences.
- {$projectname}: Represents the project or application folder that will be automatically created.

## GitOps Kubernetes Manifest 
The configuration repository houses the Kubernetes manifests, forming a pivotal aspect of GitOps principles:
- The configuration repository mirrors the manifests present in the Kubernetes cluster.
- Any commit made to the configuration repository triggers the deployment of the new version of the files to the cluster (to be facilitated by a defined pipeline).
- It is imperative that all configuration changes are reflected through Git commits. Ad-hoc alterations using 'kubectl' commands directly on the cluster are strictly prohibited.

### Example Setup
```hcl
  clone_gitops:
    stage: manifest push
    title: cloning gitops repo
    type: git-clone
    git: "githubcodefresh12"
    arguments:
      repo: 'thinegan/codefresh-gitops'
      revision: 'master'
    when:
      branch:
        only:
          - master

  change_manifest:
    stage: manifest push
    title: "Update staging manifest"
    image: "mikefarah/yq:latest"
    working_directory: "${{clone_gitops}}"
    commands:
      - mkdir -p ${{APP_NAME}}
      - cp -f app-template/application.yaml ${{APP_NAME}}
      - NAME="${{APP_NAME}}" yq e -i '.metadata.name = strenv(NAME)' ${{APP_NAME}}/application.yaml
      - NAME="${{APP_NAME}}" yq e -i '.metadata.labels.name = strenv(NAME)' ${{APP_NAME}}/application.yaml
      - PROJECT="${{APP_PROJECT}}" yq e -i '.spec.project = strenv(PROJECT)' ${{APP_NAME}}/application.yaml
      - REPO="${{APP_REPO}}" yq e -i '.spec.source.repoURL = strenv(REPO)' ${{APP_NAME}}/application.yaml
      - LOC="./infra/charts/${{APP_NAME}}" yq e -i '.spec.source.path = strenv(LOC)' ${{APP_NAME}}/application.yaml
      - TAG="${{RELEASE_IMAGE_TAG}}" yq e -i '.spec.source.helm.parameters[0].value = strenv(TAG)' ${{APP_NAME}}/application.yaml
      - ELB="${{INGRESS_ELB_SG}}" yq e -i '.spec.source.helm.parameters[1].value = strenv(ELB)' ${{APP_NAME}}/application.yaml
      - ACM="${{CERT_ACM}}" yq e -i '.spec.source.helm.parameters[2].value = strenv(ACM)' ${{APP_NAME}}/application.yaml
      - ENP="${{KUBE_ENDPOINT}}" yq e -i '.spec.destination.server = strenv(ENP)' ${{APP_NAME}}/application.yaml
      - NS="${{HELM_NAMESPACE}}" yq e -i '.spec.destination.namespace = strenv(NS)' ${{APP_NAME}}/application.yaml
      - cat ${{APP_NAME}}/application.yaml
    when:
      branch:
        only:
          - master
```

### Prerequisites
- AWS Account Staging and Production
- Codefresh Service
- Cloudflare DNS  Service
- Synk (Security Platform)
- AWS EKS Cluster Staging and Production

## Author
Thinegan Ratnam
 - [https://thinegan.com](https://thinegan.com/)

## Copyright and License
Copyright 2024 Thinegan Ratnam

Code released under the MIT License.
