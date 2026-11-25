# 235 — Trabalhar com arquivos no Linux

**Data da prática:** 19/11/2026 | **Ambiente:** Amazon Linux 2 na EC2/Vocareum, acesso SSH pelo Debian WSL.

## Objetivo
Criar um backup compactado da pasta CompanyA, registrar data/hora do backup e mover o arquivo para IA. Este documento se baseia no roteiro Escola da Nuvem e nos resultados efetivamente compartilhados, não em uma nova execução.

## Tarefa 1 — Acesso SSH
A conexão SSH à instância foi realizada com uma chave PEM temporária e o usuário ec2-user.

## Tarefa 2 — Criar um backup
A estrutura fornecida pelo laboratório e conferida no terminal foi:

~~~text
CompanyA/
  Employees/Schedules.csv
  Finance/Salary.csv
  HR/Assessments.csv
  HR/Managers.csv
  IA/
  Management/Promotions.csv
  Management/Sections.csv
  SharedFolders/
~~~

Comandos executados:

~~~bash
pwd
ls -R CompanyA
tar -csvpzf backup.CompanyA.tar.gz CompanyA
ls
~~~

A saída da criação do backup listou os diretórios e arquivos processados. O comando ls confirmou que backup.CompanyA.tar.gz foi criado ao lado de CompanyA em /home/ec2-user.

O tar reúne arquivos em um arquivo de backup; a opção -z aplica gzip. A ordem de opções apresentada é a utilizada no próprio roteiro, que funcionou no laboratório.

## Tarefa 3 — Registrar o backup
~~~bash
cd /home/ec2-user/CompanyA
touch SharedFolders/backups.csv
echo "$(date '+%d/%m/%Y %H:%M:%S'), backup.CompanyA.tar.gz" | sudo tee SharedFolders/backups.csv
cat SharedFolders/backups.csv
~~~

**Saída real confirmada:**

~~~text
19/11/2026 23:47:21, backup.CompanyA.tar.gz
~~~

O comando date forneceu o horário, echo construiu a linha e tee exibiu e gravou o resultado. O CSV foi criado depois de gerar o arquivo compactado, então não integra o conteúdo daquele backup inicial.

## Tarefa 4 — Mover o backup
~~~bash
pwd
mv ../backup.CompanyA.tar.gz IA/
ls . IA
~~~

**Saída real:**

~~~text
/home/ec2-user/CompanyA
.:
Employees  Finance  HR  IA  Management  SharedFolders

IA:
backup.CompanyA.tar.gz
~~~

O caminho .. aponta para o diretório pai. Por isso, mv localizou o backup um nível acima de CompanyA e o colocou em IA.

## Referência rápida
| Comando | Função |
| --- | --- |
| pwd | Localiza o diretório atual |
| ls -R | Exibe arquivos e pastas recursivamente |
| tar -csvpzf | Cria um arquivo tar compactado com gzip |
| cd | Entra em outro diretório |
| touch | Cria arquivo vazio |
| date | Mostra data e hora formatadas |
| echo | Exibe ou constrói uma linha de texto |
| tee | Mostra a saída e escreve em arquivo |
| cat | Mostra conteúdo de arquivo |
| mv | Move arquivo ou pasta |

## Resultado
**Confirmado:** backup criado, CSV escrito com a data real e arquivo movido para IA. As quatro tarefas práticas do laboratório foram realizadas. O encerramento do Vocareum não foi comprovado no histórico.

**Segurança:** nenhum IP temporário, chave SSH ou credencial foi publicado.