<img width="60" src="images/ansible.svg"/>

# Automate Kubernetes Deployment 

In this project, we are going to deploy ngix server with ansible into a EKS cluster. 

## Step 1: Create an EKS cluster 
we can use `Terraform` to provision an EKS cluster. In explain [here](https://github.com/hotiaDiallo/terraform-playground/tree/provision-eks) how to create eks clsuter using terraform. 

We can also use `eksctl` to create the cluster 
```
eksctl create cluster --name k8s-cluster --region eu-west-3 --nodegroup-name devops-node --node-type t2.micro --nodes 2 --nodes-min 1 --nodes-max 3
```

Follow this [link](https://github.com/hotiaDiallo/devops-java-maven-app/tree/pipeline-with-aws-eks-and-ecr) for more details

## Step 2: Create our playbook and run it 
The goal is to create a namespace and deploy the application into that namespace. For that we are going to use 

### Task 1 : install pre-requisites
Since we are using `kubernetes.core.k8s` module, we need to install some pre-requisites before deploying our application

```
- name: install pre-requisites
      ansible.builtin.pip:
        name:
          - openshift
          - pyyaml
          - kubernetes 
```

![Image](/images/eks.png)

### Task 2 : Create namespace my-app-ns

```
- name: Create a k8s namespace
      kubernetes.core.k8s:
        name: my-app-ns
        api_version: v1
        kind: Namespace
        state: present
        kubeconfig: ~/devops/projects/sample-k8s-app/kubeconfig-myapp-cluster
```

### Task 3 : Create namespace my-app-ns

```
- name: Deploy nginx app 
      kubernetes.core.k8s:
        src: ~/devops/projects/sample-k8s-app/nginx.yaml
        state: present
        kubeconfig: ~/devops/projects/sample-k8s-app/kubeconfig-myapp-cluster
        namespace: my-app-ns
```

Run the playbook

```
ansible-playbook deploy-app-to-k8s.yaml
```

## Notes 

we use the `kubeconfig` for ansible to be able to connect to our cluster. 
f not provided, and no other connection options are provided, the Kubernetes client will attempt to load the default configuration file from `~/.kube/config`. Can also be specified via `K8S_AUTH_KUBECONFIG` environment variable.