# Infraestrutura por Código - Aplicação em Python para gerenciar infraestrutura construída com Terraform

- **Aluno:** Bernardo Cunha Capoferri
- **Curso:** Engenharia da Computação
- **Semestre:** 6
- **Contato:** bernardocc@al.insper.edu.br
- **Ano:** 2022

Agradecimentos a Francisco Pinheiro Janela por permitir o uso deste template.

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

#### Serviço web High Availability

- O serviço de alta disponibilidade na web é fixo, podendo ser alterado um booleano que define se este será criado ou não e o par de chaves que será associado às instâncias.

- Para verificar o funcionamento do sistema de serviço na web é necessário acessar o DNS do load-balancer que é um dos *outputs* do Terraform, ou a partir do *dashboard* da AWS. A página deverá conter o ip interno da instancia e um valor de *nonce*

- Para verificar a criação de uma nova instância automática acesse a instância via SSH, usando seu par de chaves e rode o comando:
``` sh 
$ ssh ubuntu@<ip da sua instância>
$ stress --cpu 8 --timeout 300
```
Espere entre 4-5 minutos e verifique se outra instância foi criada.

# Tutorial do terraform:

## O que é Terraform?

O terraform é um software de *Infrastructure as Code* (IaC) cuja a linguagem faz com que a criação de infraestrutura em diferente provedores de serviços na nuvem como AWS, Azure e mais seja fácil, rápido e mais importante, replicável.

## Estrutura do Terraform

O Terraform contém no mínimo um arquivo main.tf, ondem devem ser definido as seguintes configurações:

- os provedores dos serviços na nuvem
- A região na qual a infraestrutura será construida

### Recursos

Com estes items definidos é possível começar a criar sua infraestrutura, para isso o Terraform tem os `resources`, que efetivamente são as partes dela, como instâncias, grupos de segurança, VPCs etc...

Exemplo de um `resource` de instânica aws:
``` terraform 
resource "aws_instance" "app_server" {
    ami             = "ami-123"
    instance_type   = "t2.micro"
    associate_public_ip_address = true
    key_name = "bernardo"

    subnet_id       = aws_subnet.subnet_projeto.id
    vpc_security_group_ids = ["example_sec_group"]

    tags = {
        Name        = "exemplo"
    }
}
``` 


Com os recursos, o programador consegue criar elementos fixos, e assim como linguagens de código tradicional ele permite a criação de recursos com variáveis o que acaba sendo útil para criar recursos dinâmicamente.

Exemplo de um `resource` de instânica aws com uso de variáveis:
``` terraform 
resource "aws_instance" "app_server" {
    ami             = var.image_id
    instance_type   = var.host_type
    associate_public_ip_address = true
    key_name = "bernardo"

    subnet_id       = aws_subnet.subnet_projeto.id
    vpc_security_group_ids = [var.sec_group]

    tags = {
        Name        = var.image_name
    }
}
```

### Variáveis

Mas ao contrário de linguagens tradicionais, não é possível simplesmente declarar variáveis, para isso é necessário defini-lás com o bloco `variable` na qual o programador pode definir o seu tipo, assim como descrição e checagem do que foi inserido.

Exemplo de um `resource` de variavel:
``` terraform
variable "image_id" {
    description = "example description"
    type = string
}
```

As variáveis por padrão são definidas sem valor padrão e como sendo obrigatórias, portanto se não forem preenchidas em um arquivo `.tfvars`, que pode ser definido tanto no formato `.json` quanto padrão do *Terraform*, será necessário inseri-las manualmente

Exemplo de um arquivo `.tfvars.json` para definições de variáveis:
``` json
{
    "image_id" : "ami-1234"
}
```

### Iteração

O Terraform também oferece mecanismos para criar vários recursos a partir de um bloco só, nominalmente os construtores `count` e `for_each`.

O `count` quando definido cria o número especificado de cópias de um recurso, então por exemplo, quando queremos criar 5 instâncias com as mesmas configurações é melhor usar o `count`.

Exemplo de um `resource` de instânica aws com uso de `count`:
``` terraform
resource "aws_instance" "app_server" {
    count           = 5
    ami             = "ami-1234"
    instance_type   = "t2.micro"
    associate_public_ip_address = true
    key_name = "bernardo"

    subnet_id       = aws_subnet.subnet_projeto.id
    vpc_security_group_ids = [aws_security_group.custom_sec_group[lookup(var.security_group_vars, each.value.security_group_name, null).name].id]

    tags = {
        Name          = "instance-${count.index}"
    }
```

Se quer maior flexibilidade na criação de novos recursos a partir de um mesmo bloco de código, é recomendado o uso do `for_each`,  que quando definido a partir de uma lista ou dicionário cria um objeto chamado `each`
que itera sobre os valores de cada um dos índicies ou chaves.

Exemplo de um `resource` de instânica aws com uso de `for_each`:
``` terraform
resource "aws_instance" "app_server" {
    for_each        = var.instance_vars
    ami             = each.value.image_id
    instance_type   = each.value.host_type
    associate_public_ip_address = true
    key_name = "bernardo"

    subnet_id       = aws_subnet.subnet_projeto.id
    vpc_security_group_ids = [aws_security_group.custom_sec_group[lookup(var.security_group_vars, each.value.security_group_name, null).name].id]

    tags = {
        Name          = each.value.image_name
    }
}
```

### Rquisições para Providers

Caso queira informações da plataforma de serviço na web para qual está desenvolvendo seu código, o módulo `data` irá te auxiliar. Com ele é possível requisitar informações externas ao seu ambiente local, como as imagens de uma região específica, ou recursos externos a sua infraestrutura.

Exemplo de um módulo `data` para obter as zonas de disponibilidade de uma região:
``` terraform
data "aws_availability_zones" "available" {
    state = "available"
}
```

### Outputs

O Terraform também permite formatar as respostas de seu programa após este ter sido aplicado por meio do módulo `output`, Assim é possível imprimir informações que você só pode ser capaz de obter após a criação efetiva de um recurso, como por exempo, o ip designado para uma instância.

Exemplo de um módulo `output` para lsitar as senhas de acesso dos usuários criados:
``` terraform
output "password" {
    description = "Password of the created users"
    value = [for password in aws_iam_user_login_profile.login_bernardo : password]
}
```

## Agora é sua vez!

Tendo uma ideia de como funcionam e como usar as ferramentas fornecidas pelo *Terraform* agora você pode começar a criar seu próprio IaC, Vasculhe a [documentação oficial]((https://registry.terraform.io/providers/hashicorp/aws/latest/docs)) e os [tutoriais](https://developer.hashicorp.com/terraform/tutorials) oferecidos tanto pela HashiCorp assim como terceiros para desenvolver a aplicação que deseja!