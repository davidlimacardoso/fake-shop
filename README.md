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

## Variável de Ambiente
DB_HOST	=> Host do banco de dados PostgreSQL.

DB_USER => Nome do usuário do banco de dados PostgreSQL.

DB_PASSWORD	=> Senha do usuário do banco de dados PostgreSQL.

DB_NAME	=>	Nome do banco de dados PostgreSQL.

DB_PORT	=>	Porta de conexão com o banco de dados PostgreSQL.
