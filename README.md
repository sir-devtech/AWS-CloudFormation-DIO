# 🚀 Implementando sua Primeira Stack com AWS CloudFormation

Repositório criado como entregável do desafio prático da [DIO (Digital Innovation One)](https://www.dio.me) — Trilha **Fundamentos de Cloud com AWS**.

---

## 📋 Sobre o Projeto

Este projeto documenta a experiência prática de implementação de Stacks no **AWS CloudFormation**, utilizando templates em formato **YAML** para provisionar recursos na nuvem de forma automatizada e repetível.

> **AWS CloudFormation** é um serviço que permite modelar, provisionar e gerenciar recursos da AWS usando código (Infrastructure as Code — IaC). Com templates JSON ou YAML, você define a infraestrutura desejada e o CloudFormation cuida da criação, atualização e exclusão dos recursos.

---

## 🎯 Objetivos de Aprendizagem

- [x] Compreender o conceito de Infrastructure as Code (IaC)  
- [x] Criar e implantar Stacks no AWS CloudFormation  
- [x] Provisionar instâncias EC2 via template YAML  
- [x] Criar Security Groups (Firewall) com regras de acesso  
- [x] Configurar buckets S3 com Lifecycle Policy  
- [x] Documentar processos técnicos de forma clara e estruturada  
- [x] Utilizar o GitHub para compartilhamento de documentação técnica  

---

## 🗂️ Estrutura do Repositório

```
AWS-CloudFormation-DIO/
├── README.md
├── templates/
│   ├── 01-EC2.yaml               # EC2 simples
│   ├── 02-Apache-Webserver.yaml  # EC2 com Apache via UserData
│   ├── 03-Firewall.yaml          # EC2 + Security Group (Firewall)
│   └── 04-S3-Buckets.yaml        # Buckets S3 com Lifecycle Policy
└── images/
    ├── 01-cloudformation-console.png
    ├── 02-stack-ec2-criada.png
    ├── 03-instancia-ec2-rodando.png
    ├── 04-stack-firewall.png
    └── 05-instancias-multiplas.png
```

---

## 📄 Templates CloudFormation

### 1️⃣ EC2 Simples — `01-EC2.yaml`

Cria uma instância EC2 básica (`t2.micro`) na zona `us-east-1a`.

```yaml
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0ed9277fb7eb570c9
      InstanceType: t2.micro
```

**Como implantar:**
1. Acesse o console AWS → CloudFormation → **Create Stack**  
2. Selecione **Upload a template file** → faça upload de `01-EC2.yaml`  
3. Nomeie a stack (ex: `EC2-Lab01`) → avance e clique em **Create stack**  

---

### 2️⃣ Apache Webserver — `02-Apache-Webserver.yaml`

Cria uma instância EC2 com:
- **Security Group** permitindo HTTP (porta 80) e SSH (porta 22)
- **UserData** para instalar e iniciar o Apache automaticamente

```yaml
Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permite acesso HTTP e SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum install -y httpd
          systemctl start httpd
```

---

### 3️⃣ Firewall (Security Group) — `03-Firewall.yaml`

Cria uma instância EC2 protegida por um Security Group configurado como firewall:
- **Permite** apenas tráfego HTTP (porta 80)
- **Bloqueia** SSH (porta 22) — acesso restrito

```yaml
Resources:
  FirewallSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Firewall - Permite apenas HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
```

---

### 4️⃣ Buckets S3 com Lifecycle — `04-S3-Buckets.yaml`

Cria dois buckets S3 privados com políticas de ciclo de vida:

| Bucket | Finalidade | Expiração |
|--------|-----------|-----------|
| `meu-projeto-backups` | Armazenar backups | 15 dias |
| `meu-projeto-logs` | Armazenar logs | 30 dias |

Características:
- **DeletionPolicy: Retain** — dados preservados ao deletar a stack  
- **AccessControl: Private** — acesso restrito  
- **LifecycleConfiguration** — remoção automática de arquivos antigos  

```yaml
Resources:
  S3BackupBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      AccessControl: Private
      LifecycleConfiguration:
        Rules:
          - ExpirationInDays: 15
            Status: Enabled
```

---

## 🔑 Conceitos-Chave

| Conceito | Descrição |
|----------|-----------|
| **Stack** | Conjunto de recursos AWS criados a partir de um template |
| **Template** | Arquivo JSON/YAML que descreve os recursos da stack |
| **AWS::EC2::Instance** | Recurso para provisionar instâncias EC2 |
| **AWS::EC2::SecurityGroup** | Firewall virtual para controle de tráfego |
| **AWS::S3::Bucket** | Bucket de armazenamento de objetos |
| **DeletionPolicy** | Define o comportamento do recurso ao excluir a stack |
| **UserData** | Script executado na inicialização da instância |
| **LifecycleConfiguration** | Regras automáticas de gestão de ciclo de vida no S3 |
| **Outputs** | Valores exportados pela stack após a criação |
| **!Ref** | Função intrínseca para referenciar outros recursos |
| **!GetAtt** | Retorna atributos de um recurso (ex: IP público) |

---

## 🛠️ Pré-requisitos

- Conta na AWS (Free Tier disponível)
- Acesso ao console AWS: [console.aws.amazon.com](https://console.aws.amazon.com)
- Permissões para CloudFormation, EC2 e S3

---

## 🚀 Passo a Passo — Como Implantar uma Stack

### Via Console AWS

1. Acesse **AWS Console** → pesquise por **CloudFormation**  
2. Clique em **Stacks** → **Create Stack** → *With new resources*  
3. Em **Prepare template**, selecione **Choose an existing template**  
4. Em **Template source**, selecione **Upload a template file**  
5. Faça upload do arquivo `.yaml` desejado  
6. Clique em **Next** → informe o nome da Stack  
7. Clique em **Next** → **Next** → **Submit**  
8. Acompanhe os eventos na aba **Events** até o status `CREATE_COMPLETE`  

### Via AWS CLI

```bash
# Implantar stack EC2
aws cloudformation create-stack \
  --stack-name EC2-Lab01 \
  --template-body file://templates/01-EC2.yaml \
  --region us-east-1

# Verificar status
aws cloudformation describe-stacks \
  --stack-name EC2-Lab01 \
  --region us-east-1

# Deletar stack
aws cloudformation delete-stack \
  --stack-name EC2-Lab01 \
  --region us-east-1
```

---

## 📸 Screenshots

As capturas de tela do processo estão na pasta [`/images`](./images/).

| # | Descrição |
|---|-----------|
| 01 | Console AWS CloudFormation — tela inicial |
| 02 | Stack EC2-Lab01 criada com status CREATE_COMPLETE |
| 03 | Instância EC2 rodando no painel EC2 |
| 04 | Stack com Firewall (Security Group) configurada |
| 05 | Múltiplas instâncias rodando simultaneamente |

---

## 💡 Aprendizados e Insights

- **IaC elimina o trabalho manual:** ao invés de configurar cada recurso pelo console, um único template reproduz toda a infraestrutura em segundos
- **Templates são reutilizáveis:** o mesmo arquivo YAML pode criar ambientes idênticos de dev, staging e produção
- **Pagamento por uso:** você paga apenas pelos recursos criados (as stacks em si não têm custo)
- **DeletionPolicy é essencial em produção:** sem ela, deletar a stack pode apagar dados importantes
- **UserData automatiza a configuração:** scripts de inicialização eliminam a necessidade de acessar a instância via SSH para instalar software
- **Security Groups como firewall:** permitem controle granular de portas e origens de tráfego, sendo a primeira camada de defesa da instância

---

## 📚 Recursos e Referências

- [Documentação Oficial AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [AWS CloudFormation — Tipos de Recursos](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [DIO — Digital Innovation One](https://www.dio.me)
- [GitHub Markdown Guide](https://docs.github.com/en/get-started/writing-on-github)

---

## 👨‍💻 Autor

Desenvolvido como parte do desafio prático da **DIO** — Trilha Fundamentos de Cloud com AWS.

[![GitHub](https://img.shields.io/badge/GitHub-sir--devtech-181717?style=flat&logo=github)](https://github.com/sir-devtech)
