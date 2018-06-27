---
title: Lab 10 - Create Image Builder
workshops: trusted_software_supply_chain
workshop_weight: 20
layout: lab
---
# Containers

Containers are an important foundation for your application in building a Trusted Software Supply Chain.  You want a secure and blessed golden container image that your application will inherit security controls from.

Containers are built using a layered approach. For example, to create a container of a Java web application, you could do so in multiple layers: the OS, the JVM, the web server, and the code itself.

<img src="../images/golden_images.png" width="900" />

We can incorporate CVE and vulnerability scanning against images in an automated fashion.  Any image change is scanned to improve the inherited security of your application.  We have partners such as Black Duck and Twistlock that do container image scanning.  Also, as a result of Red Hat's acquisition of CoreOS, we now offer Quay as an enterprise-grade registry that does vulnerability scanning.

We can also cryptographically sign your image so you know your container is running with a verified container image.

# Start with Quality Parts

Red Hat has a container registry that provides certified Red Hat and third-party container images that will be the foundation of your container images.  Our Registry also has a health index of the image so you know the state of the image.

<img src="../images/rh_container_catalog.png" width="900" />

# Add Create Image Builder Stage

Next we will add the Create [Image Builder][1] Stage into your pipeline.

<img src="../images/pipeline_build_image.png" width="900" />

This step will create a new build.  We will be leveraging a trusted JBoss EAP 7 container.

The golden image will will be using for our applications is jboss-eap70-openshift:1.5.  Again, you'll want a hardened, secured, patched and up to date container image as a foundation for your application.

Append the text below to your text file or into the YAML/JSON field for tasks-pipeline in the OpenShift Console.


```
            stage('Create Image Builder') {
                when {
                  expression {
                    openshift.withCluster() {
                      openshift.withProject(env.DEV_PROJECT) {
                        return !openshift.selector("bc", "tasks").exists();
                      }
                    }
                  }
                }
                steps {
                  script {
                    openshift.withCluster() {
                      openshift.withProject(env.DEV_PROJECT) {
                        openshift.newBuild("--name=tasks", "--image-stream=jboss-eap70-openshift:1.5", "--binary=true")
                      }
                    }
                  }
                }
              }
```
[1]: https://docs.openshift.com/container-platform/3.9/architecture/core_concepts/builds_and_image_streams.html