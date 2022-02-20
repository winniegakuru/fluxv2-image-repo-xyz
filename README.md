# fluxv2-image-repo-xyz
-- Dont mind project name... 
 <p> I had to do several installations before I could get it right...
 <p> So dont beat yourself up if there are a few things you got wrong.

## Pre-requisites:
1. Kubernetes Cluster
2. Flux CLI
3. Git repo
4. Kubernetes resource (Deployments/Pods/services)

## What we will cover
In this walk-through, we will cover basic approach to get your fluxcd-v2 up and running in your cluster. 
 <p> For more advanced production setup like Multi tenancy, Team based setup... Kindly see Fluxcd
official support on that from:
1. https://github.com/fluxcd/flux2-kustomize-helm-example
2. https://github.com/fluxcd/flux2-multi-tenancy

> Also Note  <p>We are using FluxCd-V2 ( With the use of Gitops ToolKit) learn more about the 5 controllers supported by the toolkit from here :https://fluxcd.io/docs/components/ 

## Objectives
1. Bootstrap Flux on a Kubernetes Cluster
2. Deploy a sample application using Flux
3. Customize application configuration through Kustomize patches

### Environment setup:
#### Kubernetes Cluster
 <p> To try it out on your local machine
 > I will recommend Kubernetes kind (https://kind.sigs.k8s.io/docs/user/quick-start/).
 <p> Although it can be a bit frustrating if your local env is not compartible
   
#### For my windows distribution
   ```go install sigs.k8s.io/kind@v0.11.1``` command worked for me!!!
 
  ![image](https://user-images.githubusercontent.com/17796294/146667928-85b705b0-25b4-47d0-b1cf-1b3f2e9a8a97.png)

 
 ![image](https://user-images.githubusercontent.com/17796294/146666565-9b9f87c7-61a3-4dbc-80e1-c2605444245d.png)
 
 #### For a cloud solution
  > might consider setting up a Minikube on AWS EC2 ( don worry it will take you less than 10 minutes).
  <p> Follow the article https://www.radishlogic.com/kubernetes/running-minikube-in-aws-ec2-ubuntu/ by Radish Logic to set up you Minikube on EC2.
   
#### A few things have changed that is not include in the article:
  1. The VM specifications given will not work and in the middle of the setup you may be required to adjust storage and other specs...
  I used ```14 GB (gp2) storage, 4 GB ram, 4vCPUs, t2.xlarge instant type...```
   Yea I know, I was a bit extra... but you can adjust the spec provided minikube starts and deploy.
  
  2. After installing Minikube, you will have to install conntrack by running 
   ```sudo apt-get install -y conntrack```
  
  3. Dont forget aws Key-pair... you will use this to access your server...
  
 
 ### FLUX CLI installation
 <p> You will need flux Cli to be able to install fluxcd controllers:
 <p> Follow the guide below. Depending on your OS distribution.
 <p> https://fluxcd.io/docs/installation/#install-the-flux-cli

 ![image](https://user-images.githubusercontent.com/17796294/146667960-632200fe-9308-47c6-a0da-84c5fc50b901.png)

 ```Choco install flux```
 
  ### GIT Repo
  <p> Fluxcd works very well with most of the common git providers: 
  <p> Follow the guide as per your provider : https://fluxcd.io/docs/installation/
   
 > I will demo using both Gitlab and Github git providers since there is no much changes
   
##### GitHub 
   ```flux bootstrap github --components-extra=image-reflector-controller,image-automation-controller --owner=<git username eg winniegakuru> --repository=fluxv2-image-repo1   --branch=main   --path=clusters/my-cluster  --read-write-key   --personal```
   
##### Gitlab
    ```flux bootstrap gitlab --components-extra=image-reflector-controller,image-automation-controller --owner=fluxcd2  --repository=Fluxcdv2   --branch=master   --path=clusters/my-cluster   --token-auth --personal```
    
> The above commands (depending on your provider) will install FluxCd toolkit including automatic image sync pod, deploy read/write key, and commit it to the specified branch.
   
 ## Kubernetes resources
 <p> Once that has been installed successfully, flux is now synchronizing  your repo and cluster.
     
  > I recommend  following fluxcd official page (https://fluxcd.io/docs/guides/image-update) to learn more on setting up automatic image update from the regitry...
     A few things like image policy, the registry to watch, interval of scanning and most importantly your image tagging.
     
  #### Other wise
  
   <p>   Here are the kubernetes files that you should commit to your repo and watch FluxCd do magics
    
```apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: wingakmso1@gmail.com
        name: fluxcdbot
      messageTemplate: '{{range .Updated.Images}}{{println .}}{{end}}'
    push:
      branch: main
  update:
    path: ./clusters/my-cluster
    strategy: Setters ```
 
    <p> --- flux-system-automation.yaml
    
```apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: podinfo
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: podinfo
  policy:
    semver:
      range: 5.0.x ```
     
--- pod-policy.yaml

```apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  image: ghcr.io/stefanprodan/podinfo
  interval: 1m0s ```
 <p>--- pod-registry.yaml

#### Conclussion
  
This is just a basic setup using flux...
for more complex Production setup, you will need to visit the official github repo: 
https://github.com/fluxcd/flux2-kustomize-helm-example
https://github.com/fluxcd/flux2-multi-tenancy

#### Also note:
  
 <p> We have not reviewed fluxcd V2 Notification controller and Monitoring.. 
   
