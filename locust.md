# Locust Assimilation
>CURRENT VERSION : 0.14.6
>[LOCUST SOURCE DOC](https://docs.locust.io/en/0.14.6/)

The aim for this repository is to provide a configuration based approach to be able to run scalable performance tests against your application . And more importantly , to develop this as a community going forward (as that is how this has started off)

>PLEASE NOTE : This is a community driven tooling approach . So there is no paticular owner product team to pass feataure requests . You can raise it as issues on the repository and be part of the community in trying to improve the product. 


Intended to test

- APIs
- Server Side for Web applications
- Any HTTP/(s) website
- Test messaging / Event based systems. 

## Aim


- To arrive at performance profile of the application (Scalable profile,  throughput profile)
- Amount of concurrent users systems can handle

The tool used (as obviously from ^ - https://github.com/locustio/locust)


# Getting Started

## Pre-Requisites

- bash
- [jq](https://stedolan.github.io/jq/)
- terraform
- az cli
- kubectl
- helm (2 and 3 ,depends on what you have running on the server. Recommend Helm3)
- vault cli
- python 3 (lots of what is here wont work on 2)
- terraform cli


### Docker images used

- [python3](https://github.com/DigitalInnovation/image-definitions/tree/master/docker/infrastructure/python3)
- [helm-bash](https://github.com/DigitalInnovation/image-definitions/tree/master/docker/build-infra/helm-bash)
- [locust](images/README.md)

## Quickstart 

- ### [local start](local.md)
- ### [building your own image](image.md)
- ### [Deploying at on AKS/kubernetes](kubernetes.md)
- ### [Deploying at on ACI](aci.md)
- ### [Automation and Scale](scaled_locust.md)
- ### [Metrics , Reports , Dashboard](report.md)
- ### [demo](demo.md)
- ### [Other Fancy stuff](random_explore.md)
- ### [Example Tests](samples.md#samples)

# Contribution Guidelines

# Contributers

-   Janos Nagy, [janos-mands](https://github.com/janos-mands) on Github 
-   Elizabeth Colgate, [elizcol](https://github.com/elizcol) on Github
-   Steven Gonsalvez, [stevengonsalvez](https://github.com/stevengonsalvez) on Github
-   Arun Kumar, [arunkumar-vn](https://github.com/arunkumar-vn) on Github
-   Chris Mitchelmore, [crmitchelmore](https://github.com/crmitchelmore) on Github

 
# APPENDIX
## [Tooling Rationale](tooling_eval.md)


- Summary: We chose Locusts as the tool of choice (duh !), But our journey to reach there [here](tooling_eval.md).


- <ins>Wrapper scripts in BASH ??</ins>
 All the wrapper scripts etc , we intentionally kept it in bash (so it would work in any CI tooling which almost all support out of the box)

- <ins>Why Terraform instead of ARM ?</ins>
A bit detailed to clutter this area. So kept it [here](https://gist.github.com/stevengonsalvez/d047746d11afcf4d7e19db040db4d0fe) as it is opinionated 



## When to Use what 

- On the local , it is easier to use docker to test as well (no bothering with virtual env in different places etc)
- If you use a kubernetes environment for performance , easier to use kubernetes . Also if you want proximity testing (to eliminate network as a factor)
- The problem with deploying in kubernetes is that all the locust files(and supporting files) are deployed as a configmap (which has a max size of 1MB). So if you have a locust file reading off say another config file with 20000 urls - then that will not be possible in the current setup . The best alternative is to use ACI for those scenario's to test (ACI mounts a azure file share as the hosting point for the locustfiles (The hassly alternative is to host that config file in blob and read from there , but that will need to be built in your test. We did not add it as a feature as , most of the time the locust files are well within the limit)
- With ACI , the problem is , it does not support Autoscaling and also you wont get the automatic grafana dashboard (tht you would get with kubernetes). There is an alternative built in , which creates a csv of all metrics gathered every 15 seconds(all metrics that are present in grafana), and we have an handler to download that file at the end of the test (You could go crazy with Pandas if thats your thing , or use the simple excel function of scatter plot to give you similar views to grafana). Or you could go ahead and use Azure monitor to pull the prometheus style metrics also exposed 



## Nuances and Workarounds

- There is no single docker repository to use for all of M&S . So either you build the images yourself . So the images (all standard builds) are also stored here - https://hub.docker.com/u/stevengonsalvez/ (Need to do some documentation in those registries)
- The locust docker image is a standard build (the requirements file you can find in the image folder). If in your test , you use a package that is not in that requirements file - your test will fail when using the image (you would need to add it in the requirements and rebuild the image) . Alternatively , looking to build something similar to [this](https://github.com/DigitalInnovation/image-definitions/tree/master/docker/infrastructure/python3), which will allow to add packages dynamically (although using the same base image)
- Helm chart is in the root directory . A bit annoying this (for backward compatiblity reasons) and also visibility (since everything that we need to pack into help needs to be in the chart folder). To get around from your release being cluttered, any folder that is not needed as part of the chart , add it in the .helmignore.


## Help and FAQ
### Worker's Memory Setting

Based on experience, it seems that a rule of thumb can be applied, that a worker node needs at least 2 MB RAM per each user, otherwise they can crash.

E.g. if the swarm has 200 users and 2 worker nodes, so we can assume that each node hosts 100 users, then we should allocate at least
200MB (the set value should be in format `200Mi`) per worker node in the `worker.resources.limits|requests.memory` parameter.


_why use this over standard helm_
- No way to run it with the web UI on kubernetes (with the webUI , the hatching has to be from the UI , this helm has a post trigger)
- ability to download stats in CSV as an option or post run

### Other infromation

- It is also good practice to verify that the test tool has enough resources, and it does not become the performance bottleneck itself, before starting the tests.
For memory sizing, the rule of thumb, base on experience, is: a worker node needs at least 2 MB RAM per each user, otherwise they can crash.
For CPU sizing, the rule of thumb, base on experience, is: a worker node needs at least 5m CPU per each user, otherwise they will slow down and become the performance bottleneck themselves.

E.g. if the swarm has 200 users and 2 worker nodes, so we can assume that each node hosts 100 users, then we should allocate at least 200MB (the set value should be in format 200Mi) per worker node using the worker.resources.limits|requests.memory parameters and 500 millicore (the set value should be in format 500m) per worker node using the worker.resources.limits|requests.cpu parameter.

- For result accuracy . The prometheus (grafana dashboard) scrape is set to 30s (prometheus tends to fall over with smaller values and 15s is minimum), so that is the granular average . The report (cumulative) is set to 15 seconds. If you want accurate results at per invocation, you would need to use the eventhooks in the test as described [here](samples.md#event-hooks)

## Other Possible Issues
- Helm version
- kubectl version
- KUBECONFIG , in vault - not standard path for V1 kubernetes 