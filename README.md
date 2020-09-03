# Tekton Pipelines
This repository contains a collection of Tekton tasks and pipelines to kick-start new projects

## Intro
All of the pipelines here only work properly with GitHub webhooks. This is due to the nature of Tekton, to only provide a unique $(uid), when a webhook is used to trigger the pipeline. This could alternatively be provided manually.
A single PVC should be used, to store all build artefacts. There is currently no cleanup mechanism built in, to be able to more easily debug failed builds. It can be easily added usign the cleanup-folder.yaml task if needed.

## tekton-setup
This folder contains Roles and RoleBindings to grant needed permissions to the default ServiceAccount which is used for Tekton. In a production setup you would likely use a separate ServiceAccount. Further a GitHub Binding is included which translates from the json GitHub Webhooks are sending to Tekton Parameters. This can be ignored if a non-GitHub git repo is used.

## tekton-tasks
A collection of tekton tasks I have maintained over time.
* init-pipeline.yaml - Create folders in the workspace attached, including the unique $(uid) of this build and check out the git repo to that
* cleanup-folder.yaml - rm -rf a specific folder
* container-build-push.yaml - Build and push a container using buildah (was kaniko before, but there is an issue with microdnf in UBI images)
* gradle.yaml - Build a gradle application
* maven.yaml - Build a maven application

## pipelines
Three pipelines are available including TriggerTemplates and EventListeners. You still need to expose the routes, for example using:
* oc create route edge container-build --service=github-listener-container-build
This would create a https route with edge termination for the container build listener. The pipelines have the following intention:
* container-build-pipeline - build and push a container - the sole requirement is a "imageurl" file which contains nothing but the path and tag of the built image, and a Dockerfile which is to be built
* gradle-build-pipeline - this is an extension of the container-build-pipeline and has been tested using the gradlew wrapper from a standard Quarkus build
* maven-build-pipeline - this is an extension of the container-build pipeline and has been tested building a OpenLiberty application