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
```
├── infrastructure # configurações do terraform
└── src            # codigo da app 
```
Em um diretório, mantemos as coisas relacionadas à infraestrutura e, no segundo, mantemos um código do aplicativo.

## A essência da estrutura
Indo mais fundo na pasta "infrastructure", você encontrará "environments" e "modules". 
Dentro do "environments", temos um diretório separado para cada ambiente. No "modules", você encontra módulos Terraform importados por pelo menos dois ambientes (DRY).

Essa é a essência dessa estrutura e o que define por ela ser tão boa. 
Você pode ficar tentado a usar uma abordagem orientada à configuração do Terraform, mas, dependendo da entrada, ela gera resultados diferentes. 
Se você tem três ambientes semelhantes (pre, pro, rev), por que não criar um módulo Terraform e chama-lo com uma configuração armazenada em um arquivo json? 
Essa ideia pode ser influenciada pelo capítulo de "configuration" da metodologia Twelve-Factor, mas está relacionada a um código de aplicativo (independente do ambiente), 
não a um código de infraestrutura (basicamente descreve ambientes). 
É assim que a implantação do ambiente pré pode ser executada.

```bash
terraform apply -var-file="pre.tfvars.json"
```

Se você usar uma abordagem orientada à configuração, mais cedo ou mais tarde acabará com muitas declarações condicionais e outros hacks bizarros. 
O Terraform é uma ferramenta destinada a criar descrições declarativas de sua infraestrutura, portanto, não introduza imperatividade. 
Ao preparar módulos Terraform separados para cada ambiente, vemos as coisas como elas se parecem na realidade. Temos um mapeamento direto e explícito entre o código e um estado do Terraform. 
Reduzimos o WTFs/minute. 
A solução proposta terá mais linhas de código, mas ainda deixará espaço para reutilização de código. Não se esqueça dos módulos!

“Qualquer tolo pode escrever um código que um computador possa entender. 
 Bons programadores escrevem código que os humanos podem entender.” 
 — por Martin Fowler

Apenas uma observação sobre os workspaces. Não o use para diferenciar ambientes. [Mesmo os autores não recomendam isso](https://www.terraform.io/language/state/workspaces#when-to-use-multiple-workspaces) . Depois, apresentarei um bom caso de uso para workspaces.

## Conta secundaria da AWS
Como é possível não precisar reautenticar (ou alterar um valor de AWS_PROFILE) quando estou alternando um contexto de contas da AWS?

```bash
cd infraestructure/environments/pre 
$ terraform apply 
# Recursos implantados na conta AWS de teste $ cd ../../../infrastructure/environments/pro $ terraform apply 
# Recursos implantados na conta AWS de produção # E simplesmente funciona
```
Em qualquer config.tf contidos nos módulo raiz dos environments, você encontrará a seguinte bloco:

```tf
provider "aws" {
  # ...

  assume_role {
    role_arn = "arn:aws:iam::${var.aws_account_id}:role/AssumableAdmin"
  }

  # ...
}
```




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

