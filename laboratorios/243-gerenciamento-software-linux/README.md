# 243 — Gerenciamento de software no Linux

**Data:** 25/11/2026 | **Ambiente:** Amazon Linux 2 na EC2 temporária do Vocareum, região us-west-2, acesso via Debian WSL.

## Objetivo
Atualizar pacotes com YUM, instalar e reverter uma transação do Apache pelo histórico do YUM e configurar a AWS CLI v2 para consultar atributos de uma EC2.

> **Segurança:** durante o exercício, as credenciais temporárias da AWS foram compartilhadas em capturas/conversa. **Nenhuma chave de acesso, chave secreta, token ou arquivo de credenciais foi publicado aqui.** Essas credenciais não devem ser reutilizadas; recomenda-se renová-las/invalida-las quando possível.

## Tarefa 1 — Acesso por SSH
A sessão remota usando ec2-user foi confirmada. O diretório de trabalho foi validado com:

~~~bash
pwd
~~~

**Saída real:**

~~~text
/home/ec2-user/companyA
~~~

## Tarefa 2 — Atualizar o Linux com YUM

### Consultar atualizações
~~~bash
sudo yum -y check-update
~~~
O YUM retornou sem listar atualizações disponíveis.

### Atualizações de segurança
~~~bash
sudo yum update --security
~~~

**Saída real:**

~~~text
No packages needed for security; 0 packages available
No packages marked for update
~~~

### Atualizar os demais pacotes
~~~bash
sudo yum -y upgrade
~~~

**Saída real:**

~~~text
No packages marked for update
~~~

### Instalar Apache (httpd)
~~~bash
sudo yum install httpd -y
~~~

**Resultado real:** Complete! — instalação concluída.

## Tarefa 3 — Histórico e reversão da instalação
### Identificar a transação
~~~bash
sudo yum history list
~~~

**Saída real resumida:**

~~~text
ID | Command line     | Date and time    | Action(s) | Altered
 1 | install httpd -y | 2026-11-25 23:29 | Install   | 11
~~~

### Detalhar o histórico
~~~bash
sudo yum history info 1
~~~

**Resultados confirmados:** Transaction ID 1, Return-Code: Success, Command Line: install httpd -y. O YUM listou 11 pacotes, incluindo httpd-2.4.71 e dependências como apr, apr-util, httpd-tools e mod_http2.

### Desfazer a instalação
~~~bash
sudo yum -y history undo 1
~~~

**Saída real:** o Apache e as dez dependências da instalação anterior foram removidos; a operação terminou com **Complete!**.

O YUM armazena o histórico de transações e pode reverter instalações; em produção, essa operação requer análise de dependências e possível impacto.

## Tarefa 4 — Instalar a AWS CLI v2
### Verificar Python e pip
~~~bash
python3 --version
pip3 --version
~~~

**Saída real:**

~~~text
Python 3.7.16
pip 20.2.2 from /usr/lib/python3.7/site-packages/pip (python 3.7)
~~~

### Baixar instalador oficial
~~~bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
~~~

**Resultado:** download de aproximadamente 72,7 MB, concluído em 100%.

### Extrair e instalar
~~~bash
unzip awscliv2.zip
sudo ./aws/install
~~~

**Saída real do instalador:**

~~~text
You can now run: /usr/local/bin/aws --version
~~~

### Testar ajuda
~~~bash
aws help
~~~

O manual abriu e foi fechado usando a tecla q.

**Nota de estudo:** o instalador da AWS CLI v2 usado aqui é um ZIP oficial; ele não foi instalado usando pip.

## Tarefa 5 — Configurar AWS CLI e consultar EC2

### Configuração padrão
~~~bash
aws configure
~~~

Os campos de chave de acesso foram deixados em branco neste passo, enquanto a região foi definida como **us-west-2** e a saída como **json**, conforme o roteiro.

### Credenciais temporárias do laboratório
Foi editado o arquivo **~/.aws/credentials** com o Nano, seguindo a estrutura abaixo; os campos contêm somente exemplos ilustrativos:

~~~ini
[default]
aws_access_key_id=EXEMPLO_NAO_REAL
aws_secret_access_key=EXEMPLO_NAO_REAL
aws_session_token=EXEMPLO_NAO_REAL
~~~

Foram utilizados Ctrl+O, Enter e Ctrl+X para salvar e sair. Nenhum valor real foi registrado neste documento.

**Boas práticas:** manter o arquivo de credenciais restrito ao usuário autorizado, evitar compartilhar capturas e não adicionar credenciais ao Git. Quando possível, utilizar funções IAM com credenciais temporárias em vez de chaves copiadas.

### Identificar a instância
No console EC2 foi encontrada a instância de laboratório **Command Host**, em execução, do tipo **t3.micro**, na região us-west-2.

### Consultar via AWS CLI
O comando abaixo ilustra a operação realizada; o ID temporário foi substituído por um placeholder:

~~~bash
aws ec2 describe-instance-attribute --instance-id <ID_DA_INSTANCIA> --attribute instanceType
~~~

**Resposta confirmada, com o ID omitido por discrição:**

~~~json
{
  "InstanceId": "<ID_OMITIDO>",
  "InstanceType": {
    "Value": "t3.micro"
  }
}
~~~

A resposta comprova que a AWS CLI foi instalada, configurada e conseguiu consultar um atributo real da instância.

## Referência rápida
| Comando | Finalidade |
| --- | --- |
| pwd | Identifica o diretório atual |
| yum check-update | Consulta atualizações disponíveis |
| yum update --security | Aplica correções de segurança disponíveis |
| yum upgrade | Atualiza os pacotes |
| yum install httpd -y | Instala Apache |
| yum history list | Lista histórico de transações |
| yum history info 1 | Detalha a transação 1 |
| yum history undo 1 | Tenta reverter a transação 1 |
| python3 --version | Verifica Python |
| pip3 --version | Verifica pip |
| curl ... -o awscliv2.zip | Baixa o instalador |
| unzip awscliv2.zip | Extrai o instalador |
| sudo ./aws/install | Instala AWS CLI v2 |
| aws help | Exibe a ajuda da CLI |
| aws configure | Configura região e formato |
| aws ec2 describe-instance-attribute | Consulta atributos da instância |

## Resultado
**Confirmado:** verificação do YUM sem atualizações pendentes; instalação de 11 pacotes e reversão; versões de Python/pip; instalação da AWS CLI; configuração e retorno JSON com o tipo t3.micro.

**Não confirmado:** encerramento do ambiente no Vocareum (etapas 50–51).

O exercício mostra o papel do gerenciador de pacotes, do histórico de instalações e da AWS CLI na administração de infraestrutura.