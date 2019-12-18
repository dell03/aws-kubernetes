# AWS Kubernetes

AWS Kubernetes is a Kubernetes cluster deployed using [Kubeadm](https://kubernetes.io/docs/admin/kubeadm/) tool. It provides full integration with AWS. It is able to handle ELB load balancers, EBS disks, Route53 domains and other AWS resources.

<!-- TOC depthFrom:2 -->

- [Updates](#updates)
- [Prerequisites and dependencies](#prerequisites-and-dependencies)
- [Configuration](#configuration)
    - [Using multiple / different subnets for workers nodea](#using-multiple--different-subnets-for-workers-nodea)
- [Creating AWS Kubernetes Cluster](#creating-aws-kubernetes-cluster)
- [Deleting AWS Kubernetes Cluster](#deleting-aws-kubernetes-cluster)
- [Addons](#addons)
- [Custom Addons](#custom-addons)
- [Tagging](#tagging)
- [Frequently Asked Questions](#frequently-asked-questions)
    - [How to access the Kubernetes Dashboard](#how-to-access-the-kubernetes-dashboard)

<!-- /TOC -->

## Updates

* *18.12.2019* Update to Kubernetes 1.17.0
* *24.11.2019* Update to Kubernetes 1.16.3
* *27.10.2019* Update to Kubernetes 1.16.2
* *6.10.2019* Update to Kubernetes 1.16.1
* *21.9.2019* Update to Kubernetes 1.16, update addons and Calico
* *24.8.2019* Update to Kubernetes 1.15.3, fix Ingress RBAC
* *7.8.2019* Update to Kubernetes 1.15.2
* *27.7.2019* Update to Kubernetes 1.15.1, upgrade addons and move to Terraform 0.12
* *9.6.2019* Update to Kubernetes 1.14.3
* *26.5.2019* Update to Kubernetes 1.14.2
* *17.4.2019* Update to Kubernetes 1.14.1
* *31.3.2019* Update to Kubernetes 1.14.0, Ingress 0.23.0, External DNS 0.5.12, Calico 3.6.1
* *2.3.2019* Update to Kubernetes 1.13.4 ([CVE-2019-1002100](https://github.com/kubernetes/kubernetes/issues/74534))
* *3.2.2019* Update to Kubernetes 1.13.3
* *19.1.2019* Update to Kubernetes 1.13.2
* *28.12.2018* Update Kubernetes Dashboard to 1.10.1
* *17.12.2018* Update to Kubernetes 1.13.1 and Calico 3.4.0
* *8.12.2018:* Update to Kubernetes 1.13.0, added storage class for `st1` HDD disks, Ingress 0.21.0 and Cluster Autoscaler 1.13.0
* *1.12.2018:* Update to Kubernetes 1.12.3 and External DNS 0.5.9
* *11.11.2018:* Fix error when updating ASG launch-configurations [#20](https://github.com/scholzj/terraform-aws-kubernetes/issues/20)
* *10.11.2018* Update to Kubernetes 1.12.2, Calico 3.3 and addons (Dashboard 1.10.0, Heapster 1.5.4, Ingress 0.20.0, External DNS 0.5.8, Cluster Autoscaler 1.12.1)
* *28.6.2018:* Fix error when disabling already disabled SE Linux ([#1](https://github.com/scholzj/terraform-aws-minikube/pull/1))
* *23.6.2018:* Update to Kubernetes 1.10.5
* *8.6.2018:* Update to Kubernetes 1.10.4
* *27.5.2018:* Update to Kubernetes 1.10.3 and Cluster Autoscaler 1.2.2
* *29.4.2018:* Update to Kubernetes 1.10.2
* *18.4.2018:* Update to Kubernetes 1.10.1
* *31.3.2018:* Update to Kubernetes 1.10.0, update Calico networking and update Kubernetes Dahsboard, Cluster Autoscaler, Ingress and Heapster addons
* *24.3.2018:* Update to Kubernetes 1.9.6
* *17.3.2018:* Update to Kubernetes 1.9.4
* *4.3.2018:* Fix issues with Cluster Autoscaler not scaling down nodes
* *11.2.2018:* Update to Kubernetes 1.9.3 and Cluster Autoscaler to 1.1.1
* *29.1.2018:* Add `kubernetes.io/cluster/my-kubernetes` tag also to the master subnet
* *22.1.2018:* Update Calico to 3.0.1
* *22.1.2018:* Update to Kubernetes 1.9.2, Ingres 0.10.0 and Dashboard 1.8.2
* *6.1.2018:* Update to Kubernetes 1.9.1
* *17.12.2017:* Update to Kubernetes 1.9.0, update Dashboard, Ingress, Autoscaler and Heapster dependencies
* *8.12.2017:* Update to Kubernetes 1.8.5
* *1.12.2017:* Fix problems with incorrect Ingress RBAC rights
* *28.11.2017:* Update addons (Cluster Autoscaler, Heapster, Ingress, Dashboard, External DNS)
* *23.11.2017:* Update to Kubernetes 1.8.4
* *9.11.2017:* Update to Kubernetes 1.8.3
* *4.11.2017:* Update to Kubernetes 1.8.2
* *14.10.2017:* Update to Kubernetes 1.8.1
* *30.9.2017:* Update to Kubernetes 1.8
* *28.9.2017:* Split into module and configuration; update addon versions
* *2.9.2017:* Update Kubernetes and Kubeadm to 1.7.5
* *22.8.2017:* Update Kubernetes and Kubeadm to 1.7.4
* *30.8.2017:* New addon - Fluentd + ElasticSearch + Kibana

## Prerequisites and dependencies
AWS Kubernetes deployes into an existing VPC / public subnet. If you don't have your VPC / subnet yet, you can use [this](https://github.com/scholzj/aws-vpc) configuration to create one. To deploy AWS Kubernetes there are no other dependencies apart from [Terraform](https://www.terraform.io). Kubeadm is used only on the EC2 hosts and doesn't have to be installed locally.

## Configuration

The configuration is done through Terraform variables. Example *tfvars* file is part of this repo and is named `example.tfvars`. Change the variables to match your environment / requirements before running `terraform apply ...`.

| Option | Explanation | Example |
|--------|-------------|---------|
| `aws_region` | AWS region which should be used | `eu-central-1` |
| `cluster_name` | Name of the Kubernetes cluster (also used to name different AWS resources) | `my-aws-kubernetes` |
| `master_instance_type` | AWS EC2 instance type for master | `t2.medium` |
| `worker_instance_type` | AWS EC2 instance type for worker | `t2.medium` |
| `ssh_public_key` | SSH key to connect to the remote machine | `~/.ssh/id_rsa.pub` |
| `master_subnet_id` | Subnet ID where master should run | `subnet-8d3407e5` |
| `worker_subnet_ids` | List of subnet IDs where workers should run | `[ "subnet-8d3407e5" ]` |
| `min_worker_count` | Minimal number of worker nodes | `3` |
| `max_worker_count` | Maximal number of worker nodes | `6` |
| `hosted_zone` | DNS zone which should be used | `my-domain.com` |
| `hosted_zone_private` | Is the DNS zone public or private | `false` |
| `addons` | List of addons which should be installed | `[ "https://..." ]` |
| `tags` | Tags which should be applied to all resources | see *example.tfvars* file |
| `tags2` | Tags in second format which should be applied to AS groups | see *example.tfvars* file |
| `ssh_access_cidr` | List of CIDRs from which SSH access is allowed | `[ "0.0.0.0/0" ]` |
| `api_access_cidr` | List of CIDRs from which API access is allowed | `[ "0.0.0.0/0" ]` |

### Using multiple / different subnets for workers nodea

In order to run workers in additional / different subnet(s) than master you have to tag the subnets with `kubernetes.io/cluster/{cluster_name}=shared`. For example `kubernetes.io/cluster/my-aws-kubernetes=shared`. During the cluster setup, the bootstrapping script will automatically add these tags to the subnets specified in `worker_subnet_ids`.

## Creating AWS Kubernetes Cluster

To create AWS Kubernetes cluster, 
* Export AWS credentials into environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
* Apply Terraform configuration:
```bash
terraform apply --var-file example.tfvars
```

## Deleting AWS Kubernetes Cluster

To delete AWS Kubernetes cluster, 
* Export AWS credentials into environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
* Destroy Terraform configuration:
```bash
terraform destroy --var-file example.tfvars
```

## Addons

Currently, following addons are supported:
* Kubernetes dashboard
* Heapster for resource monitoring
* Storage class for automatic provisioning of persisitent volumes
* External DNS (Replaces Route53 mapper)
* Ingress
* Autoscaler
* Logging with Fluentd + ElasticSearch + Kibana

The addons will be installed automatically based on the Terraform variables. 

## Custom Addons

Custom addons can be added if needed. For every URL in the `addons` list, the initialization scripts will automatically call `kubectl -f apply <Addon URL>` to deploy it. The cluster is using RBAC. So the custom addons have to be *RBAC ready*.

## Tagging

If you need to tag resources created by your Kubernetes cluster (EBS volumes, ELB load balancers etc.) check [this AWS Lambda function which can do the tagging](https://github.com/scholzj/aws-kubernetes-tagging-lambda).

## Frequently Asked Questions

### How to access the Kubernetes Dashboard

The Kubernetes Dashboard addon is by default not exposed to the internet. This is intentional for security reasons (no authentication / authorization) and to save costs for Amazon AWS ELB load balancer.

You can access the dashboard easily fro any computer with installed and configured `kubectl`:
1) From coomand line start `kubectl proxy`
2) Go to your browser and open [http://127.0.0.1:8001/ui](http://127.0.0.1:8001/ui)
