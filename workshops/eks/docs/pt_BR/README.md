# Workshop de Amazon EKS

Neste workshop, exploraremos várias maneiras de configurar VPC, ALB, EC2 Kubernetes DataPlane Nodes e Amazon Elastic Kubernetes Service (EKS).

# Começando

# Provisionando seu cluster EKS

Para o provisionamento do cluster, vamos usar o [**EKS Blueprints**](https://aws.amazon.com/blogs/containers/bootstrapping-clusters-with-eks-blueprints/). EKS Blueprints é uma coleção de módulos de infraestrutura como código (IaC) que ajuda a configurar e implantar clusters EKS consistentes e com addons instalados. Você pode usar EKS Blueprints para inicializar facilmente um cluster EKS com addons do Amazon EKS, bem como uma ampla variedade de addons populares de código aberto, incluindo Prometheus, Karpenter, Nginx, Traefik, AWS Load Balancer Controller, Fluent Bit, Keda , Argo CD e muito mais. O EKS Blueprints também ajuda a implementar controles de segurança relevantes necessários para operar cargas de trabalho de várias equipes no mesmo cluster.

### Etapa 1: clone o repositório usando o comando abaixo

```bash
git clone https://github.com/aws-samples/latam-containers-roadshow.git
```

### Etapa 2: execute o Terraform INIT

Inicialize um diretório de trabalho com arquivos de configuração

```bash
cd ~/environment/latam-containers-roadshow/workshops/eks/terraform/
terraform init
```

### Etapa 3: executar o Terraform PLAN

Verifique os recursos criados por esta execução

```bash
terraform plan
```

### Passo 4: Finalmente, Terraform APPLY

para criar recursos

```bash
terraform apply --auto-approve
```

## Configurar o kubectl e testar o cluster

Os detalhes do cluster EKS podem ser extraídos da saída do terraform ou do Console AWS para obter o nome do cluster. O comando a seguir atualiza o kubeconfig em sua máquina local onde você executa comandos kubectl para interagir com seu cluster EKS.

### Etapa 5: execute o comando update-kubeconfig

`~/.kube/config`arquivo é atualizado com detalhes do cluster:

```bash
aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
```

### Etapa 6: Liste todos os worker nodes executando o comando abaixo

```bash
kubectl get nodes
```


## Crie um aplicativo de exemplo para testar o CA

Implantaremos um aplicativo nginx de exemplo com um ReplicaSet de 1 pod.

```bash
cat <<EoF> ~/environment/nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        service: nginx
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 500m
            memory: 512Mi
EoF

kubectl apply -f ~/environment/nginx.yaml

kubectl get deployment/nginx-to-scaleout
```

### Validando nosso teste


```bash
kubectl get pods -l app=nginx -o wide
```

Verifique a [Console de gerenciamento EC2 AWS](https://console.aws.amazon.com/ec2/home?#Instances:sort=instanceId)para confirmar se os grupos do Auto Scaling estão ok. 

Ou verifique usando`kubectl`

```bash
kubectl get nodes,pods
```

### Limpe o ambiente

```bash
kubectl delete -f ~/environment/nginx.yaml