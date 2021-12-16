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
