# Infraestrutura por Código - Aplicação em Python para gerenciar infraestrutura construída com Terraform

- **Aluno:** Bernardo Cunha Capoferri
- **Curso:** Engenharia da Computação
- **Semestre:** 6
- **Contato:** bernardocc@al.insper.edu.br
- **Ano:** 2022


# Como Operar com o meu programa:

## Pré-requisitos

Para começar a operar com o meu programa, sigam o passo a passo de instalação indicado no readme do [meu repositório](https://github.com/bert799/ProjetoCompNuvem) no github.

## Como usar

Ao iniciar o programa pela primeira vez, o usuário se depara com a escolha entre a a criação de recursos para a infraestrutura, podendo, nesse caminho, criar instâncias, security groups e habilitar a criação de um web-server **HA** (High Availability), Listar os recursos presentes na infraestrutura, onde o usuário pode verificar o estado desta antes de aplicá-la efetivamente, Apagar os recursos em sua infraestrutura e trocar a região na qual esta construindo sua infraestrutura.

Para facilitar, a árvore de navegação abaixo, pode trazer mais facilidade para o usuário se encontrar pelo programa:

```
├── Create new infrastructure
│   ├── Instance
│   ├── Security Group
│   ├── Security Group rule
│   ├── User
│   ├── High-availability web-service
│   ├── Back
│   
├── List infrastructure
│   ├── Instance
│   ├── Security Group
│   ├── User
│   ├── High-availability web-service
│   ├── Change region
│   ├── Back
│ 
├── Delete infrastructure
│   ├── Instance
│   ├── Security Group
│   ├── Security Group rule
│   ├── User
│   ├── High-availability web-service
│   ├── Back
│
├── Change Region
│
├── Apply
│
├── Exit
```

Em sua maioria, as instruções de uso de cada uma das funcionalidades está autocontida no próprio programa, mas existem algumas considerações que podem ser feitas:

#### Instâncias

- Ao Criar uma Instância, o usuário deve escolher o tipo de instância que deseja criar, e o nome da instância que deseja criar. O nome da instância deve ser único, ou seja, não pode haver duas instâncias com o mesmo nome na mesma região. O usuário também tem a opção de adicionar um Security group anteriormente definido a instância, caso contrário um padrão será criado.


#### Security Groups

- Ao Criar um Security Group, o usuário deve escolher o nome do Security Group que deseja criar. O nome do Security Group deve ser único, ou seja, não pode haver dois Security Groups com o mesmo nome na mesma região. Além disso, poderá ser criado quantas regras de Ingress e Egress que o usuário desejar, se atentando somente à liberação de todas as portas para não correr riscos de segurança.

- Só será possível deletar um Security Group se ele não estiver sendo usado, ou seja, se quiser deletá-lo, deve remover todas as dependências dele nas instâncias da região.


#### Usuários

- Ao Criar um Usuário, o usuário deve escolher o nome do Usuário que deseja criar. O nome do Usuário deve ser único, ou seja, não pode haver dois Usuários com o mesmo nome na mesma região. Além disso, a senha aparecerá somente na criação deste, então guarde-a em um local seguro antes de continuar.

- Ao deletar um Usuário, o usuário não poderá ser recuperado, e todas as configurações de políticas atreladas também serão descartadas, mas as políticas em si ainda continuarão existindo.

# Tutorial do terraform:

## Começando

Para seguir esse tutorial é necessário:

- **Tecnologias:** Terraform, Python
- **Documentos:** [Terraform AWS Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

## O que é Terraform?

O terraform é um software de *Infrastructure as Code* (IaC) cuja a linguagem faz com que a criação de infraestrutura em diferente provedores de serviços na nuvem como AWS, Azure e mais seja fácil, rápido e mais importante, replicável.

## Estrutura do Terraform

O Terraform contém no mínimo um arquivo main.tf, ondem devem ser definido as seguintes configurações:

- os provedores dos serviços na nuvem
- A região na qual a infraestrutura será construida

Com estes items definidos é possível começar a criar sua infraestrutura, para isso o Terraform tem os **resources**, que efetivamente são as partes dela, como instâncias, grupos de segurança, VPCs etc...

??? Exemplo
    Exemplo de um **resource** de instânica aws:
    ``` python 
    resource "aws_instance" "app_server" {
        ami             = "ami-123"
        instance_type   = "t2.micro"
        associate_public_ip_address = true
        key_name = "bernardo"

        subnet_id       = aws_subnet.subnet_projeto.id
        vpc_security_group_ids = ["example_sec_group"]

        tags = {
            Name          = "exemplo"
        }
    }
    ``` 


Com os recursos, o programador consegue criar elementos fixos, e assim como linguagens de código tradicional ele permite a criação de recursos com variáveis o que acaba sendo útil para criar recursos dinâmicamente.

??? Exemplo
    Exemplo de um **resource** de instânica aws com uso de variáveis:
    ``` python 
    resource "aws_instance" "app_server" {
        ami             = var.image_id
        instance_type   = var.host_type
        associate_public_ip_address = true
        key_name = "bernardo"

        subnet_id       = aws_subnet.subnet_projeto.id
        vpc_security_group_ids = [var.sec_group]

        tags = {
            Name          = var.image_name
        }
    }
    ```

Mas ao contrário de linguagens tradicionais, não é possível simplesmente declarar variáveis, para isso é necessário defini-lás com o bloco **variable** na qual o programador pode definir o seu tipo, assim como descrição e checagem do que foi inserido.