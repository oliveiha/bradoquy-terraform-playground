## Requisitos
Vamos revisar quais casos de uso a estrutura proposta abrangerá:

AWS como provedor de nuvem. Isso também pode ser aplicado a outros, pois o Terraform oferece suporte a muitos provedores diferentes.
Configuração de várias contas da AWS. Uma conta para cargas de trabalho de produção e outra para fins de teste.
Recursos cruciais (por exemplo, banco de dados) implantados manualmente e sob demanda.
Ambientes dinâmicos de curta duração para dar suporte a aplicativos para filiais.
Compartilhamento e referência de recursos criados por outra configuração do Terraform.
** Desenvolvimento local sem atrito.
Capacidade de implantar a partir de uma máquina local sem um fluxo de autenticação complexo.
Atuar no contexto de uma determinada conta da AWS assumindo uma função.
Estados do Terraform armazenados em uma conta da AWS separada.

## Contexto
O diagrama da infraestrutura gerenciada pelo repositório é mostrado abaixo. Conforme mencionado, temos duas contas da AWS nas quais nossa infraestrutura opera. Você provavelmente está familiarizado com essa configuração, pois é bastante comum. Existe algum tipo de tópico que transmite eventos que podem ser consumidos por muitas filas e processados ​​por um aplicativo. O resultado disso precisa ser armazenado em armazenamento persistente, que no nosso caso é uma tabela do DynamoDB.
![Environment](images/01.png)

## Os recursos são atribuídos a um determinado ambiente:

com (comum). Conjunto de componentes compartilhados por muitos. No exemplo, temos um tópico game-score no AWS SNS que pode ter muitos consumidores.

profissional (produção)

pré (pré-produção/encenação). Um ambiente para fins de teste. Atua como um portão antes da produção. Espelha o ambiente de produção.

rev (revisão). Ambientes dinâmicos de curta duração. Gira sob demanda. Podemos ter muitos deles. Imita o ambiente de pré-produção.

## Estrutura proposta
```
.
├── infrastructure
│   ├── environments
│   │   ├── com
│   │   ├── pre
│   │   ├── pro
│   │   │   ├── config.tf
│   │   │   ├── db
│   │   │   │   ├── config.tf
│   │   │   │   ├── main.tf
│   │   │   │   ├── outputs.tf
│   │   │   │   └── variables.tf
│   │   │   ├── main.tf
│   │   │   ├── outputs.tf
│   │   │   └── variables.tf
│   │   └── rev
│   └── modules
│       ├── app
│       │   ├── main.tf
│       │   ├── outputs.tf
│       │   └── variables.tf
│       ├── db
│       └── queue
├── package.json
└── src
    └── index.js
```    
Antes de começarmos a dividir a ideia em fatores primos, vamos verificar como é simples e explícito implantar no pré e no pro a partir de uma máquina local.

```bash
$ cd infraestrutura/ambientes/pre 
$ terraform init 
$ terraform apply 
$ cd ../../../infrastructure/environments/pro 
$ terraform init 
$ terraform apply
```

Isso funciona para sistemas bastante complexos à medida que o usamos em nosso projeto. 
Achei essa facilidade de implantação (e de reversão se algo der errado) muito agradável 😊.

## Vamos analisar a estrutura de pastas e partir do topo.

├── infrastructure # Terraform configurations
└── src            # An app code

Em um diretório, mantemos as coisas relacionadas à infraestrutura e, no segundo, mantemos um código do aplicativo.

## A essência
Indo mais fundo na pasta "infrastructure", você encontrará "environments" e "modules". 
Dentro do "environments", temos um diretório separado para cada ambiente. No "modules", você encontra módulos Terraform importados por pelo menos dois ambientes (DRY).



# Deploy

Deploying temporary review environment using Terraform workspaces:

```
$ tfenv install
$ tfenv use
$ cd infrastructure/environments/rev
$ terraform init
$ terraform workspace new foo-bar-1
$ terraform apply
$ terraform destroy
$ terraform workspace select default
$ terraform workspace delete foo-bar-1
```

