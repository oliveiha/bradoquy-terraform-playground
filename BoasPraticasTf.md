## Ferramentas indicadas
*terraform fmt*: comando integrado do Terraform para aplicar convenções de codificação recomendadas
*tfswitch*: ferramenta útil para gerenciar e alternar entre as versões do Terraform em sua estação de trabalho
*tflint*: plugin usado para impor as melhores práticas (como convenções de nomenclatura)
*tfsec*: análise estática de seus modelos do Terraform para detectar possíveis problemas de segurança (como senhas codificadas permanentemente)
*terratest*: biblioteca de testes automatizada para validar se a infraestrutura funciona corretamente nesse ambiente, fazendo solicitações HTTP, chamadas de API, conexões SSH, etc.
*terraform_docs*: pacote usado para gerar documentação para módulos Terraform
*checkov*: análise de código estático para ferramentas IaC, que vem com uma coleção bem definida de verificações de práticas recomendadas para AWS e Azure, bem como suporte para regras personalizadas
*pre-commit*: coleção de git hooks que forçam/lembram o desenvolvedor de fazer certas ações (por exemplo, fmt, lint, docs) antes de enviar o código do Terraform

## tenha um bloco "terraform" em sua configuração
O bloco terraform é um componente importante, mas opcional, que configura alguns dos comportamentos do próprio Terraform. Eu gosto de pensar nisso como um conjunto de pré-requisitos para você poder executar o código, bem como onde você configura seu estado remoto e bloqueia o arquivo se estiver usando.
Depois de usar o Terraform por um tempo e trabalhar com versões diferentes, você entenderá por que é importante ter um bloco de código que verifica essencialmente seu ambiente de desenvolvedor para garantir que você tenha tudo configurado corretamente. 
```
terraform {
  required_version = "~> 0.13"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3"
    }
  }
  backend "s3" {
    bucket = "mystatefilebucket"
    key    = "terraformstate.tfstate"
    dynamodb_table = "mylockfile"
  }
}
```
**O "~>" é um truque muito legal para dizer ao Terraform que qualquer versão dentro dessa versão principal é válida e eu recomendo fazer isso para o required_version**

## Entenda seu arquivo de estado e o que tem nele
O arquivo de estado é, simplesmente, o local onde está armazenado o estado atual e a configuração de todos os recursos gerenciados pelo Terraform e é sem dúvida o componente mais importante do sua pipeline do Terraform, portanto, você deve tratá-lo como tal. 
[Cuidado com dados sensiveis](https://www.terraform.io/language/state/sensitive-data)

## Comente as variáveis ​​e crie sempre Outputs blocks
Enquanto você está desenvolvendo, pode parecer perfeitamente claro o que variable name {} significa .Depois de ir além de uma estrutura de arquivo simples, aproveitando as saídas de módulos, outros arquivos de estado e outros objetos de dados, você perceberá rapidamente que é importante entender exatamente quais variáveis ​​vêm de qual local e ter o máximo de verificações para garantir que os problemas sejam encontrados cedo. O Terraform torna isso facil com a validação de variáveis.

Considere o exemplo abaixo, que é uma variável para uma Amazon Machine Image for AWS. Não há razão para que ele não funcione com apenas variable image_id {}e você possa passar qualquer valor que desejasse para lá e, desde que existisse, passaria na verificação de validação durante o terraform plan. Claro, quando você fosse aplicar a configuração, ela falharia em tempo de execução se não fosse uma referência a uma ami real. Podemos colocar algumas informações aqui para fazer algumas verificações durante o estágio de plan que nos ajudarão a detectar erros no início do pipeline, além de fornecer mais informações ao consumidor sobre o que é esperado, não é incomum que ami seja um mapa baseado na região para exemplo.

```
variable image_id {
  type        = string
  description = "The id of the machine image (AMI) to be used."

  validation {
    condition     = length(var.image_id) == 21 && 
                    substr(var.image_id, 0, 4) == "ami-"
    error_message = "The image_id value must be a valid AMI id."
  }
}
```
Você pode usar funções integradas do Terraform para validação, existem muitas delas, incluindo regex.
[Funções Integradas ](https://www.terraform.io/language/functions)

## Integre seu ambiente com um pipeline antecipadamente
É somente quando você começa a colaborar com outras pessoas e começa a aproveitar o controle de origem e os módulos externos que pode ficar realmente complicado. Meu conselho é que você deve configurar seu ambiente em uma pipeline o mais rápido possível.

## Manje de funções e meta-argumentos da HCL
Embora em suas raízes a HashiCorp Configuration Language (HCL) seja uma coleção de provedores, recursos, variáveis ​​e saídas, a maneira como você os conecta pode exigir um pouco mais de sutileza do que o anunciado. Isso pode ser devido à maneira como a API do vendor retorna um valor, a ordem em que os recursos são criados ou talvez o provedor Terraform ainda esteja sendo desenvolvido, mas tenha certeza de que voce se perguntará por que saporra não funciona?
Insira funções e meta-argumentos, alguns dos quais já falamos anteriormente. A HCL tem um conjunto bastante padrão de ferramentas para gerenciar tipos de objetos (strings, listas e assim por diante) que você pode usar para manipular as entradas para serem o que você deseja que elas sejam. Alguns dos melhores exemplos de como fazer esse tipo de coisa podem ser encontrados no Terraform Registry.

```
resource "aws_default_security_group" "this" {
  count = var.create_vpc && var.manage_default_security_group ? 1 : 0

  vpc_id = aws_vpc.this[0].id

  dynamic "ingress" {
    for_each = var.default_security_group_ingress
    content {
      self             = lookup(ingress.value, "self", null)
      cidr_blocks      = compact(split(",", lookup(ingress.value, "cidr_blocks", "")))
      ipv6_cidr_blocks = compact(split(",", lookup(ingress.value, "ipv6_cidr_blocks", "")))
      prefix_list_ids  = compact(split(",", lookup(ingress.value, "prefix_list_ids", "")))
      security_groups  = compact(split(",", lookup(ingress.value, "security_groups", "")))
      description      = lookup(ingress.value, "description", null)
      from_port        = lookup(ingress.value, "from_port", 0)
      to_port          = lookup(ingress.value, "to_port", 0)
      protocol         = lookup(ingress.value, "protocol", "-1")
    }
  }
```
Nesse caso, há uma condicional na criação do recurso usando a count para definir se o recurso será criado, seguido por um número dinâmico de regras de entrada com base em uma lista de mapas de string. O for_each percorre cada mapa na lista e executa pesquisas nos valores de determinadas chaves e cria várias regras de entrada com alguns padrões aceitáveis ​​no caso de a chave não existir, além de lidar com vários valores (é para isso que splite compact servem ). A beleza de algo assim é que ele permite que você mantenha todas as alterações de configuração em seus arquivos de variáveis ​​de ambiente, em vez de ter que fazer muitas alterações frequentes em seu código de infraestrutura subjacente.
Apenas saiba que existem essas e muitas outras maneiras de fazer com que o Terraform forneça o resultado que você deseja, sem ter que copiar e colar blocos do seu código subjacente.

[Meta-argumentos do Terraform] ([count]https://www.terraform.io/language/meta-arguments/count, [lifecycle](https://www.terraform.io/language/meta-arguments/lifecycle), [relationships_on](https://www.terraform.io/language/meta-arguments/depends_on) e [for_each](https://www.terraform.io/language/meta-arguments/for_each) )

[Funçoes Terraform](https://www.terraform.io/language/functions)

## Rode o comando terraform com ’var-file’
Com o var-file, você consegue gerenciar ambientes de forma mais fácil e evita uma longa lista de pares chave-valor para as variáveis ( -var chave=valor), transformando o comando em algo quase ilegível. Por exemplo:

```
$ cat config/dev.tfvars
name = "dev-stack"
s3_terraform_bucket = "dev-stack-terraform"
tag_team_name = "hello-world"
$ terraform plan -var-file=config/dev.tfvars
```

## Sempre use backends para gerenciar os tfstates
Todas as vezes que recursos são criados via ‘terraform apply’, o arquivo tfstate é criado ou modificado. Se recursos são modificados ou novo recursos são adicionados, o tfstate é atualizado.

Em um time, vários membros podem modificar o código terraform. Logo, para manter o mesmo tfstate, deve-se usar um backend.

## Ative o controle de versão nos buckets dos tfstates
Sempre ative o versionamento do bucket S3 de backend. Dessa forma, você conseguirá acompanhar com mais precisão o que foi alterado (e quando) na infraestrutura. 

## Nunca commit em seu repositório o tfstate
O tfstate armazena informações de mapeamento entre os recursos que você criou e sua real infraestrutura. Comitar estes arquivos gera vários riscos, tais como, por exemplo:
Você pode estar expondo configurações de aplicação, passwords, string de conexões com bancos etc.
Você pode estar executando um tfstate obsoleto ou antigo que ficou commitado.

## Execute o Terraform em builds automatizados
Rodar código em uma ferramenta de build automatizado tem várias vantagens, que incluem ter um processo repetível e histórico de mudanças.
O conceito de build é bem útil quando aplicado ao Terraform, garantindo mais visibilidade sobre o que e quando foi alterado na infraestrutura (para auditoria ou mesmo, debug)

## Não altere o tfstate manualmente (sempre use o CLI)
Quando quiser mover ou remover algum estado, não fique tentado a se aventurar pelo ftstate. É perfeitamente possível fazer a alteração manual, mas o tfstate pode se tornar um arquivo muito complexo e caótico.
A melhor forma é utilizar o CLI do Terraform, que lhe provê comandos como ‘terraform state rm’ e ‘terraform state mv’.

## Ative o DEBUG quando precisar de troubleshooting
Às vezes, as mensagens de erro do Terraform podem ser bem incompreensíveis. Por isso, para facilitar o troubleshooting, ative o DEBUG do log do Terraform. 
Basta alterar o valor da variável de ambiente TF_LOG. Por exemplo: TF_LOG=DEBUG && terraform plan

## Use o ‘terraform import’ para importar recursos
Recursos que já foram codificados não têm a necessidade de serem refeitos. Evite retrabalho.

## Evite configurações ‘hard coded’ sempre que possível
Dentre as razões, podemos listar as várias formas de quebra de segurança e também dificuldade de editar essas configurações.
Algumas informações, como account id, account alias e region podem ser obtidas via data sources. Por exemplo:
```
data "aws_caller_identity" "current" {}
data "aws_iam_account_alias" "current" {}
data "aws_region" "current" {}

locals {
   account_id = data.aws_caller_identity.current.account_id
   account_alias = data.aws_iam_account_alias.current.account_alias
   region = data.aws_region.current.name
}
```
## Use o ‘terraform fmt’
Sempre rode ‘terraform fmt’ para manter seus arquivos de configuração formatados. Você pode inclusive colocar a execução do ‘terraform fmt’ na sua esteira de CI/CD, sempre que for fazer o merge para o master. Por exemplo:
```
terraform validate
terraform fmt -check=true -write=false -diff=true
```

## Use módulos compartilhados
Não há necessidade de ficar reescrevendo código. Por isso, utilizar módulos no terraform irá lhe poupar algumas boas horas de programação.

## Atualize sempre a versão do Terraform
A Hashicorp nem sempre segue de perto as regras de versionamento semântico. Sem contar que novas funcionalidades liberadas pelos cloud provider podem não funcionar em versões antigas do Terraform, mesmo atualizando os plugins.

## Rode o Terraform a partir de um container
Existem containers oficiais de Terraform que você pode usar. Com isso, você controla de forma mais fácil e transparente qual versão do Terraform está usando. 
Para esteira CI/CD, é extremamente recomendado usar essas imagens. Exemplo de utilização:
```
$ TERRAFORM_IMAGE=hashicorp/terraform:0.12.3
$ TERRAFORM_CMD=`docker run -ti --rm -w /app -v ${HOME}/.aws:/root/.aws -v ${HOME}/.ssh:/root/.ssh -v `pwd`:/app -w /app $TERRAFORM_IMAGE`
$ `TERRAFORM_CMD` init
$ `TERRAFORM_CMD` plan
```

## Use a variável ‘self’
A variável ‘self’, como na maioria das linguagens de programação, se refere ao próprio objeto. E, no caso do Terraform, se refere ao resource.
Logo, sempre que precisar de alguma informação do próprio objeto que se esteja manipulando, use a sintaxe self.<ATTRIBUTE>
Vale observar que a sintaxe self.<ATTRIBUTE> só é permitida dentro de provisioners. Por exemplo:
```
resource "aws_ecr_repository" "jenkins" {
   name = var.image_name
   provisioner "local-exec" {
     command = "./deploy-image.sh ${self.repository_url}"
  }
}
```

## Use plugins pré-instalados
Há uma forma de usar plugins pré-instalados no Terraform em vez de baixá-los a cada ‘terraform init’. 
Você pode usar estes plugins colocando-os no mesmo diretório do binário do Terraform ou passando a flag -plugin-dir.  
[Entenda melhor](https://learn.hashicorp.com/tutorials/terraform/automate-terraform#pre-installed-plugins) 

## Use o locking of state file
O Terraform oferece uma forma de realizar uma prevenção de execuções paralelas em cima de um mesmo tfstate. Este locking ajuda a garantir que apenas um membro do time esteja alterando configurações da infraestrutura em um mesmo momento.
Esse locking ajuda a prever conflitos, perda de dados e evita que o tfstate fique corrompido.
O DynamoDB pode ser usado como o mecanismo de locking. A tabela precisa ter uma chave chamada LockID. Por exemplo:
```
resource “aws_dynamodb_table” “terraform_state_lock” {
    name = “terraform-lock”
    read_capacity = 5
    write_capacity = 5
    hash_key = “LockID”
    attribute {
        name = “LockID”
        type = “S”
    }
}

terraform {
    backend “s3” {
        bucket = “terraformbackend”
        key = “terraform”
        region = “us-east-2”
        dynamodb_table = “terraform-lock”
    }
}
```

## Não coloque credenciais hard-coded
Colocar as credenciais no código é uma enorme quebra de segurança, principalmente se forem suas credenciais pessoais (e não de serviço). Para evitar isso, temos 2 soluções rápidas:
Environment variables:
```
export AWS_ACCESS_KEY_ID=”accesskey”
$ export AWS_SECRET_ACCESS_KEY=”secretkey”
$ export AWS_DEFAULT_REGION=”us-west-2"
$ terraform plan
```
Shared credentials file
```
provider “aws” {
    region = “sa-east-1”
    shared_credentials_file = “/Users/tf_user/.aws/creds”
    profile = “myprofile”
}
```
## Gere o README para cada módulo com inputs e outputs
Um módulo bem documentado tem muito mais chances de ser utilizado. E você nem ao menos precisa gerar a documentação manualmente. 
Uma ferramenta chamada ‘terraform-docs’ pode criar esta documentação para você. 
[Para mais detalhes](https://github.com/cytopia/docker-terraform-docs) 

## Conheça as built-in functions, aprenda algumas e use-as!
O Terraform provê várias built-in functions e algumas delas podem ser realmente muito úteis quando se está criando infraestruturas mais completas. 
Várias podem realmente valer a pena de serem estudadas. Algumas delas são: chomp, format, indent, join, regex, replace, split, trim, upper, concat, contain, list, map, sort, base64encode, abspath, basename, fileexits, dateformat, timestamp, base64sha256, md5, tobool, tonumber, tostring, cidrhost, cidrnetmask, cidrsubnet, dentre outras.

## Conheças “side-tools”
O Terraform, como qualquer ferramenta, linguagem ou framework, tem várias ferramentas de terceiros que facilitam muito sua utilização. Conheça algumas e use as que preferir. Algumas delas são, por exemplo: 
[Atlantis](https://www.runatlantis.io/)
[Terragrunt](https://terragrunt.gruntwork.io/)
[TerraSpace](https://terraspace.cloud/)
[Terraform-docs](https://github.com/cytopia/docker-terraform-docs)

## Visite o site Terraform Best Practices
Uma boa fonte de boas práticas é o site [Terraform Best Practices](https://www.terraform-best-practices.com/) . Acessá-lo periodicamente pode ser muito útil, pois ele está sempre sendo atualizado à medida que novas versões surgem.

## Terraform não é bala de prata
Só isso mesmo



