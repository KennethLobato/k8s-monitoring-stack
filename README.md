# k8s-monitoring-stack

## Prerequisitos y herramientas a tener instaladas en local con últimas versiones disponibles.
- aws cli con cuenta configurada de AWS Access Key ID y AWS Secret Access Key
- terraform
- kubectl

## Generación de cluster K8s en AWS EKS

Se parte de la plantilla de ejemplo de Terraform, disponible en el siguiente enlace:
- https://learn.hashicorp.com/tutorials/terraform/eks

```
git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
```

Se cogen los archivos del repositorio anterior necesarios para la creación del cluster EKS.

1) Creamos e inicializamos el entorno de Terraform. En la raíz de la descarga de este repositorio:

```
terraform init
```

2) Editamos el fichero de eks-cluster.tf para ajustar los worker_groups como deseemos:

```
worker_groups = [
  {
    name                          = "worker-group-1"
    instance_type                 = "t2.small"
    additional_userdata           = "echo foo bar"
    asg_desired_capacity          = 2
    additional_security_group_ids = [aws_security_group.worker_group_mgmt_one.id]
  },
  {
    name                          = "worker-group-2"
    instance_type                 = "t2.medium"
    additional_userdata           = "echo foo bar"
    additional_security_group_ids = [aws_security_group.worker_group_mgmt_two.id]
    asg_desired_capacity          = 1
  },
]
```

3) Previsualizamos el cluster de EKS a generar:

```
terraform plan
```

4) Lanzamos la creación del cluster de EKS (tarda como 10 minutos):

```
terraform apply
```

5) Configuramos el kubectl

```
aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)
```

Verificamos la correcta vinculación del nuevo cluster:
```
kubectl config get-contexts
```

Revisamos nodos y pods en ejecución:

```
kubectl get nodes -o wide

kubectl get pods -n kube-system

kubectl get pods --all-namespaces
```

6) Despliegue de Metric Server
```
wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 && tar -xzf v0.3.6.tar.gz

kubectl apply -f metrics-server-0.3.6/deploy/1.8+/

rm -rf metrics-server-0.3.6 v0.3.6.tar.gz
```

Verificamos que se ha desplegado de forma correcta:
```
$ kubectl get deployment metrics-server -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           87s
```

7) Despliegue y uso de Kubernetes Dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

Ejecutamos un proxy para trasladar a local la comunicación con el Dashboard:
```
kubectl proxy
```

Abrimos el siguiente enlace en el navegador [http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy]

Creamos un Role Binding para permitir al cluster-admin acceder al cuadro de mando:
```
kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/master/kubernetes-dashboard-admin.rbac.yaml
```

Extraemos el token para acceder al Kubernetes Dashboard:
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')
```

![Kubernetes Dashboard](./images/KubernetesDashboard.png)
