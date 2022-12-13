**Monitoring with Grafana Agent**

Grafana Agent is a lightweight telemetry collector based on Prometheus that only performs its scraping and **remote\_write** functions. Here we have a rabbitmq application in our kubernetes cluster and Grafana Agent can be used to ship rabbitmq scraped metrics to Grafana Cloud using its remote\_write feature.

**Configure Grafana Agent in the cluster**

1. In the grafana cloud, they have **kubernetes monitoring configuration**. There, we have to install the prebuilt dashboard and rules which will create some general dashboard where we can monitor our kubernetes cluster pods and nodes under different namespaces. And also by installing these will allow us to open the configuration instructions.									
1. In the instruction, we get the following configurations:
- **Grafana Agent config file:** We find the **remote\_write** credentials in the config file which can be saved in the azure key-vault if we don’t want to use them directly in the file for security reasons. By default, the ConfigMap only scrapes cluster metrics endpoints like the **/cadvisor** and **/kubelet** endpoints. We need to configure the ConfigMap to scrape **/metrics** endpoints of Pods deployed in our cluster. Read more here: <https://grafana.com/tutorials/k8s-monitoring-app/> 					
- **Manifest\_url:** Combined with service account, cluster-role, cluster-role-binding, service and statefulset yaml files for the agent. We should change the namespaces in those files to ‘grafana-ag’.			
- **Helm release for kube-state-metrics:**  Kube-state-metrics watches Kubernetes resources in your cluster and emits Prometheus metrics that can be scraped by Grafana Agent. More information: <https://github.com/kubernetes/kube-state-metrics>					
- **(Optional)**: There are also DaemonSet to tail container logs and configmap to collect traces for the kubernetes workloads. We can ship these traces to Grafana Cloud for storage and querying from our hosted Grafana instance. 

**Deploy in the project**

In order to deploy the Grafana agent in the project, a permission to read/write for the cluster is required. The repository is called ‘**evaluate-tf-serivces’** and we deploy the Grafana agent inside the ‘**infra**/**k8s\_config’** directory. As we use terraform in the project, so we deploy the grafana agent and kube-state-metrics by terraform as well.

1. **Create namespace for Grafana agent:** We create the namespace ‘**grafana-ag’** for grafana agent and we deploy everything under this namespace. Namespaces are created in k8s\_namespaces.tf and the path is: infra/k8s\_config/k8s\_namespaces.

1. **Creating a helm chart for Agent:** Here we create a helm chart for the agent called ‘**gachart**’ with all the yaml files under the ‘**infra/k8s\_config/helm**’ directory, instead of using them directly in the project.           <https://phoenixnap.com/kb/create-helm-chart>.  							                                                                                       
1. **Deploy helm releases by terraform:** A terraform file called ‘**helm\_releases**’ is used which is under the ‘*infra/k8s\_config’* to deploy the helm chart in the project. For kube-state-metrics we have ‘**ksm**’ and for grafana agent we have ‘**grafana-agent**’. The ksm chart is used from a github repository and if we want to change anything for the chart we can do as well. Here is an example:

![alt text](https://github.com/toriqul007/grafana-ag/blob/d834299cc35b75e0df8f307dcfa55d71f8be1793/pic1.png)     



1. **Variables**: 											
- **Remote\_write credentials:** We don’t wanna use the credentials directly in the file, so we save the credentials in the key vault secret and we use variables in the file instead.                                                                                 

![alt text](https://github.com/toriqul007/grafana-ag/blob/d834299cc35b75e0df8f307dcfa55d71f8be1793/pic2.png)

` `**As the credentials are static and we get it from the grafana cloud, we save it manually in the key-vault**. Here is how we use the secret from the keyvault into our file. 

![alt text](https://github.com/toriqul007/grafana-ag/blob/d834299cc35b75e0df8f307dcfa55d71f8be1793/pic3.png)

- **Other variables:** We can also use other variables in the config yaml files for grafana agent. For example grafana-agent statefulset image version. Here is how this variable is used:

![alt text](https://github.com/toriqul007/grafana-ag/blob/d834299cc35b75e0df8f307dcfa55d71f8be1793/pic4.png)



**Rabbitmq Monitoring in Grafana Cloud:**

After deploying the agent in the cluster, we can import a rabbitmq pre-built dashboard and this dashboard will automatically detect the rabbitmq application inside the cluster and monitor it by collecting the metrics. The dashboards for rabbitmq can be found here:

<https://grafana.com/grafana/dashboards/?src=ggl-s&mdm=cpc&camp=nb-rabbitmq-exact&cnt=137469178685&trm=rabbitmq+dashboard+grafana&device=c&gclid=Cj0KCQiAkMGcBhCSARIsAIW6d0A5QP0nKT5ajXnF1jVRz8prNUvf7yyZNbC-SL6C5OAZDewhMOJS3UcaAp9pEALw_wcB&search=rabbitmq>
