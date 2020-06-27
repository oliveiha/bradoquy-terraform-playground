# Introdução-Terraform

Bem-vindo ao Terraform - Introdução. 

## Usando os arquivos

Cada pasta representa um módulo do curso e é completamente independente. Em cada módulo, haverá um exemplo do arquivo * tfvars * que você usará chamado * terraform.tfvars.example *. Simplesmente atualize o conteúdo do arquivo e renomeie-o * terraform.tfvars *. Devido à natureza sensível das informações que você coloca no arquivo * tfvars *, ** não as verifique no controle de origem, especialmente em um repositório público. Alguns de nós - * leia-me * - já cometemos esse erro antes e tivemos que excluir as chaves de acesso da AWS após a pressa.

Depois de atualizar e renomear os arquivos * tfvars *, você pode executar os comandos no arquivo * m # _commands.txt *, em que * # * é o número do módulo. Certifique-se de executar os comandos no diretório ativo do módulo. Ou você pode simplesmente mexer na CLI do terraform e ver o que pode descobrir / quebrar. Se você encontrar um problema, envie-o como tal e farei o possível para corrigi-lo.

## Pares de chaves da AWS

Um dos problemas mais comuns relatados pelas pessoas é a confusão sobre os principais pares e regiões da AWS. As configurações do Terraform usam us-east-1 (N. Virginia) como região padrão. Você pode substituir essa região alterando o padrão ou enviando um valor diferente para `var.region`. O par de chaves da AWS que você usa deve ser criado na mesma região que você selecionou para implantação. Você pode criar essas chaves no AWS EC2 Console ou na AWS CLI. Se você estiver usando a CLI, o processo é muito simples.

`` console
aws configurar região definida your_region_name
aws ec2 criar-par-de-chave - nome-da-chave nome_da_chave
`` ``

A saída json incluirá uma seção KeyMaterial. Copie e cole o conteúdo da seção KeyMaterial começando com `----- BEGIN RSA PRIVATE KEY -----` e terminando com `----- END RSA PRIVATE KEY -----` em um arquivo com uma extensão .pem. Aponte a entrada * tfvars * para `private_key_path` para o caminho completo do arquivo.

Se você estiver usando o Windows, lembre-se de que as barras invertidas do caminho do arquivo precisam ser dobradas, pois a barra invertida única é o caractere de escape para outros caracteres especiais. Por exemplo, o caminho `C: \ Users \ Ned \ mykey.pem` deve ser inserido como` C: \\ Users \\ Ned \\ mykey.pem`.

## Finais de linha

Outra questão que eu descobri de tempos em tempos é que o Terraform não gosta muito do estilo do Windows de terminar uma linha com um retorno de carro (CR) e um avanço de linha (LF), comumente referido como CRLF. Se você estiver enfrentando problemas de análise estranhos, altere a linha que termina como apenas Line Feed (LF). No código VS, isso pode ser desativado clicando no CRLF no canto inferior direito e alterando-o para LF.

## DINHEIRO !!!

Um lembrete gentil sobre custo. O curso fará com que você crie recursos na AWS e no Azure. Alguns dos recursos não serão 100% gratuitos. Na maioria dos casos, tentei usar o [Free-tier] (https://aws.amazon.com/free/) quando possível, mas em alguns casos eu escolhi usar uma instância de EC2 de tamanho maior para demonstrar as possibilidades com ambientes múltiplos.

A zona DNS no Azure também não é totalmente gratuita. Você precisará comprar um domínio DNS, se ainda não tiver um, e definir o Servidor de Nomes para usar o DNS do Azure. Se você optar por um TLD de marca externa como .xyz, poderá conseguir um nome de domínio por cerca de US $ 0,99 no primeiro ano. O DNS do Azure é de cerca de US $ 0,50 por zona por mês e US $ 0,40 por milhão de consultas. No geral, você está procurando cerca de US $ 2,00 por uma zona DNS.

Ao concluir um exercício no curso, desmonte a infraestrutura. Cada arquivo de exercício termina com `terraform destroy '. Basta executar esse comando e aprovar a destruição para remover todos os recursos da AWS.

## Certificação

A HashiCorp estará lançando a certificação * Terraform Certified Associate * em um futuro próximo - dependendo de quando você estiver lendo isso, ela já estará disponível. Você pode estar se perguntando se este curso o prepara totalmente para o certificado. ** Não. ** Fazer este curso junto com o curso [Deep Dive - Terraform] (https://app.pluralsight.com/library/courses/deep-dive-terraform) sobre o Pluralsight atenderá a maior parte do aprendizado objetivos para a certificação, mas não há substituto para a execução do software por si próprio e a remoção.

Estou trabalhando em um guia de certificação com outros dois autores e fornecerei um link assim que estiver pronto. Este é um guia não oficial, mas acredito que, em conjunto com os cursos da Pluralsight, você estará em uma boa posição para fazer o exame.

## Conclusão

Espero que você goste de fazer este curso tanto quanto eu

