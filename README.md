# Fake Shop



### Gerando a Imagem da Aplicação
```bash
    docker build -t davidlimacd/fake-shop:v1 --push .
```

### Aplicar o Manifesto no k8s 

 ```bash
    kubectl apply -f k8s/deployment.yaml 
    kubectl get pod
    kubectl get svc fakeshop -o jsonpath='{.status.loadBalancer.ingress[0
    ].hostname}'
 ```

### Variável de Ambiente
DB_HOST	=> Host do banco de dados PostgreSQL.

DB_USER => Nome do usuário do banco de dados PostgreSQL.

DB_PASSWORD	=> Senha do usuário do banco de dados PostgreSQL.

DB_NAME	=>	Nome do banco de dados PostgreSQL.

DB_PORT	=>	Porta de conexão com o banco de dados PostgreSQL.

### Criar AWS Role para Autenticação da Action Github
1. Crie o Provedor de Identidade OIDC

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1 
```  

  2. Crie a Política de Permissões

- Crie um arquivo JSON com a política de permissões. Aqui está um exemplo de política que permite interagir com o EKS:
> eks-policy.json
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster",
        "eks:ListClusters",
        "eks:ListNodegroups",
        "eks:ListUpdates",
        "eks:DescribeNodegroup",
        "eks:DescribeUpdate"
      ],
      "Resource": "*"
    }
  ]
}
```

- Crie a política usando o comando AWS CLI:
```bash
aws iam create-policy \
  --policy-name GitHubActionsEKSAccessPolicy \
  --policy-document file://eks-policy.json
```

3. Crie a IAM Role com a Política de Confiança
- Crie um arquivo JSON com a política de confiança, adicione seu usuário, repositório e branch que assumirão a role:

> trust-policy.json
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<your-account-id>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:sub": "repo:<your-github-username>/<your-repo-name>:ref:refs/heads/<your-branch-name>"
        }
      }
    }
  ]
}
```

- Crie a role usando o comando AWS CLI:
```bash
aws iam create-role \
  --role-name GitHubActionsEKSRole \
  --assume-role-policy-document file://trust-policy.json
```

4. Anexe a Política de Permissões à Role
Anexe a política de permissões criada anteriormente à role:

```bash
aws iam attach-role-policy \
  --role-name GitHubActionsEKSRole \
  --policy-arn arn:aws:iam::<your-account-id>:policy/GitHubActionsEKSAccessPolicy
```
5. No novo modelo de permissão do EKS mais atual, pode ser necessário dar permissões para o Github Action aplicar sei `manifest` em seu cluster Kubernetes:

Vá em EKS > Clusters > SEU_CLUSTER > IAM access entries > Create Access Entry. Em IAM principal  selecione a Role `GitHubActionsEKSRole` > Avançar > Selecione a policy `AmazonEKSAdminPolicy` e define em qual namespace (default) > Create.

## Referências

http://imersao.devopspro.com.br

https://github.com/actions/checkout

https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/store-information-in-variables

https://github.com/aws-actions/configure-aws-credentials

https://github.com/marketplace/actions/deploy-to-kubernetes-cluster

https://github.com/marketplace/actions/docker-login

https://github.com/marketplace/actions/build-and-push-docker-images