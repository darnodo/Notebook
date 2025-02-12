---

title: "My First Lab"
cascade:
  type: docs

---

## Introduction

In this article, we will see how to install our first containerlab netlab using devpod.  
We will focus on cloud provider to host our project, why ?  
Because Network Lab can consume huge amount of ressources, and we need to be able to deploie it easily and, per financial perspectif, stop and destroy them quickly.

For that, thanks to `devpod`, `devcontainer`and `containerlab`

On this article, we are going to use a little topology shared on my [github](https://github.com/darnodo/VXLAN-EVPN).

Our main objectif here shoulb be to deploie this lab on **AWS** using devpod.

## Prerequisite

Before make it works, we have to authorized devpod to access to our AWS environment.  
We will, at the first step, create a specific IAM role to permit to DevPod to managed the instance.  
