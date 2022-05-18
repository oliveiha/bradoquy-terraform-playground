## Infraestrutura como codigo - Conceitos proncipais

´´´
1 - Por não ser um processo manual, provisiona infraestrutura como codigo para alcançar consistencia e previsibilidade.
2 - Usa arquivos de configuração (código) para definir sua infraestrutura no formato yaml, json ou hashcorp configuration.
3 - Deve ser armazenado em alguma ferramenta de versionamento de codigo.
4 - Totalmente declarativa
5 - idempotencia/consistencia
6 - Push ou Pull - Não utiliza agents para receber os estado do ambiente

Deploy automatizaddo
Ambientes consistentes
Processos repetitivos
Reutilização de componentes 
Arquitetura documentada
´´´

### Deploy de uma infraestrutura automatizada
´´´
- provisionamento de recursos
- Planning updates
- Utilização de um SCM
- Reutilização de templates
´´´

### Terraform componentes
´´´
Terraform Executable
1 - Escrito em Go

Terraform files
- Arquivos terraform geralmente na extensão .tf . Quando o terraform ve esses arquivos ele junta tudo .

Terraform plugins 
- É um executavel simples, ele utiliza varios plugins para interagir com o provider exe (aws, azure, gcp).

Terraform state
- Uma vez criado os recursos, o terraform gosta de acompanhar o que esta acontecendo, para que ele tenha 
´´´

# Existem varias maneiras de definir variaveis no terraform, uma delas é utilizando o o arquivo .tfvars
´´´
Esse arquivo defini valores para as variaveis contidas na configuração
´´´
# Comando utilizados no modulo m3
´´´
# inicializando a configuração, verifica o provider utilizado, baixa os plugins necessarios, 
terraform init
terraform validate
# terraform plan verifica os arquivos de configuração no diretorio de trabalho atual e tambem carrega as variaveis encontradas no .tfvars , o -out armazena o plano em um arquivo para quando eu realemente desajar criar os recursos.
terraform plan -out m3.tfplan
# Aplica o m3.tfplan obtido no terraform plan
terraform apply "m3.tfplan"
# terraform destroy examina o .tfstate e quais recursos foram criados e em seguida destroi tudo, lista toda a configuração e pergunta se realmente desaja destruir.
terraform destroy
´´´


### m4 - Atualizando nossa configuração com mais recursos

´´´
Infraestrtura como codigo não estaticas, elas evoluem com o tempo. E uma das coias que vai acabar fazendo com qualquer infra que implantar é implata-la ou fazer alterações/interações na infraestrutura existente para atingir novos objetivos de negocio ou atender requisitos tecnicos.
´´´

´´´
# terraform state
1 - formato .json, **nunca altere ele se não souber o que esta fazendo"
2 - É responsavel por gerenciar e manter o estado da configuração aplicada.
3 - Mapeia recursos e metadados e a versão do estado, numero de serie 
4 - Suporta bloqueio , bloqueio é uma forma do terraform sinalizar que o tfstate esta em um estado de fluxo e que ninguem mais deveria tentar fazer alteraçoes na infra ate que a atual alteração seja aplicada
5 - armazenamento local, por padrão fica alocado no diretorio onde vc executou a primeira vez o terraform apply, armazenamento remoto (aws,azure, terraform cloud, nfs )
6 - workspace, separa .tfstates em workspaces, o terraform entende que quando vc altera o contexto entre workspaces ele alterna o arquivo de estado.
7 -  Regras do terraform
     a - depois de implantar com terraform, faça todas as alterações no terraform, embora o terraform sincronize algumas das informações de volta ao planejar seu proximo conjunto de mudanças, ele não sincroniza tudo e não tem informações de nada que não esteja sobre seu gerenciamento.
´´´

´´´
# terraform planning
O processo de planejamento do terraform inspeciona e começa uma atualização. Compara o que esta no .tfstate com a configuração desejada, cria o grafico de dependencias (ex - rede antes de instancias ec2 ) analisa em que ordem os recursos devem ser criados, atualizados e excluidos. Utiliza paralelismo para criação re recursos que não dependem um do outro

### Configurando um recurso depois da criação

´´´
Sintaxe da limguagem hashcorp
Usa-se a linguagem hashcorp para valer-se do uso das (expressoes, condicionais, funcoes para manipulação de objetos ).

terraform usa blocos para definir as coisas, 
ex simples de uso de bloco:
block_tipe label_one label_two{


}

## provisionadores terraform

O terraform tem a capacidade de executar provisionadores ou seja executar configuração apos a implantação
A hashcorp preferi que vc use algo como ansible, puppet ou chef para fazer isto visto que o terraform não é capaz de rastrear o estado interno da uma instancia ec2.
Podem ser de 2 tipos (local ou remoto)
podem ser uxecutados durante a criação ou destruição de um ambiente


## Adicionando um novo provider para sua configuração

´´´
funções - as categorias mais importantes são:
1 - numericas (manipulam apenas numeros)
ex min(42,13,7)
2 - string 
ex lower("TACOS")
3 - collections (listas e mapas)
ex merge (map1, map2)

4 - filesystem

5 - ip network
ex cidrsubnet()

6 - Date and time

## configurando network


variables -

providers - 

data - 

resources - 

output - 

##

block type

functions = Tipos Numeric min(42, 16, 37)
                              String lower ("TACOS") 
                              Collection merger(map1, map2)
                              filesystem file(path)
                              IP Network cidrsubnet()
                              Date and time timestamp()



