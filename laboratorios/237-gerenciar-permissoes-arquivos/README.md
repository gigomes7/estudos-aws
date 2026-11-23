# 237--Lab — Gerenciar permissões de arquivos no Linux

## Objetivo

Praticar o gerenciamento de **proprietários, grupos e permissões de arquivos** no Linux, com `chown`, `chmod`, os modos simbólico e absoluto e a conferência dos resultados.

**Data da prática:** 23/11/2026  
**Fonte:** roteiro do **237--Lab - Gerenciar permissões de arquivo**, Escola da Nuvem / Vocareum, e saídas reais do terminal compartilhadas durante o laboratório.

## Ambiente

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Região | Oregon (`us-west-2`) |
| Instância | Amazon EC2 Linux temporária do laboratório |
| Acesso | SSH usando Debian no WSL (Windows) |
| Usuário SSH | `ec2-user` |
| Diretório trabalhado | `/home/ec2-user/companyA` |
| Ferramentas | `pwd`, `ls`, `cd`, `chown`, `vi`, `chmod` |

> A chave SSH, o IP público temporário e quaisquer credenciais não foram publicados.

## Tarefa 1 — Conectar à instância

O acesso à EC2 foi realizado por SSH a partir do Debian/WSL. A conexão foi confirmada pelo prompt `[ec2-user@... ~]$`.

## Tarefa 2 — Alterar proprietários e grupos

### 1. Localizar o diretório

Os primeiros comandos mostraram que a sessão estava em `/home/ec2-user`, com o diretório `companyA` disponível:

```bash
pwd
ls
cd /home/ec2-user/companyA
ls
```

**Atenção ao caminho:** a primeira tentativa de `sudo chown -R ljuan:HR HR` retornou `No such file or directory`, pois a sessão ainda estava em `/home/ec2-user`. Após entrar em `companyA`, o comando foi executado com sucesso.

### 2. Alterar proprietários recursivamente

```bash
sudo chown -R mjackson:Personnel /home/ec2-user/companyA
sudo chown -R ljuan:HR HR
sudo chown -R mmajor:Finance HR/Finance
```

O `-R` aplica as alterações recursivamente aos arquivos e subdiretórios. A ordem importa: primeiro é configurada a árvore principal; depois, as subpastas recebem proprietários e grupos específicos.

### 3. Conferir os resultados

```bash
ls -ld HR
ls -ld HR/Finance
ls -laR
```

**Saídas reais observadas (trechos):**

```text
drwxr-xr-x 6 ljuan HR 72 Nov 23 22:57 HR
drwxr-xr-x 2 mmajor Finance 105 Nov 23 22:57 HR/Finance
```

A listagem recursiva confirmou a seguinte estrutura:

| Local | Proprietário | Grupo |
| --- | --- | --- |
| `companyA` | `mjackson` | `Personnel` |
| `HR` e subpastas não financeiras | `ljuan` | `HR` |
| `HR/Finance` e arquivos `.csv` | `mmajor` | `Finance` |

## Tarefa 3 — Alterar permissões com chmod

### 4. Modo simbólico: adicionar escrita ao grupo

O arquivo `symbolic_mode_file` foi criado com o Vim:

```bash
sudo vi symbolic_mode_file
```

No Vim: `i` para inserir texto; `Esc` e `:wq` para salvar e sair.

```bash
sudo chmod g+w symbolic_mode_file
ls -l symbolic_mode_file
```

**Saída observada:**

```text
-rw-rw-r-- 1 root root 30 Nov 23 23:11 symbolic_mode_file
```

O `g+w` adicionou **gravação ao grupo**. Como o arquivo foi criado usando `sudo vi`, o proprietário e o grupo eram `root:root`. O comando não alterou a propriedade.

### 5. Modo absoluto: definir 764

Foi criado o arquivo `absolute_mode_file`:

```bash
sudo vi absolute_mode_file
sudo chmod 764 absolute_mode_file
ls -l absolute_mode_file
```

**Saída real observada:**

```text
-rwxrw-r-- 1 root root 20 Nov 23 23:15 absolute_mode_file
```

| Dígito | Categoria | Permissões |
| --- | --- | --- |
| `7` | Proprietário | `rwx`: leitura, gravação e execução |
| `6` | Grupo | `rw-`: leitura e gravação |
| `4` | Outros | `r--`: somente leitura |

O arquivo permaneceu com proprietário `root` e grupo `root`.

## Tarefa 4 — Atribuir proprietários às pastas Shipping e Sales

### 6. Pasta Shipping

```bash
sudo chown -R eowusu:Shipping Shipping
ls -ld Shipping
ls -laR Shipping
```

**Saída confirmada:**

```text
drwxr-xr-x 2 eowusu Shipping 6 Nov 23 22:57 Shipping
```

Houve uma tentativa inicial com `eowusu?Shipping`, que retornou `invalid user`. O comando foi corrigido usando dois-pontos (`:`) entre usuário e grupo.

### 7. Pasta Sales

```bash
sudo chown -R nwolf:Sales Sales
ls -laR Sales
```

**Saída confirmada:**

```text
Sales:
total 0
drwxr-xr-x  2 nwolf    Sales       6 Nov 23 22:57 .
drwxr-xr-x 10 mjackson Personnel 199 Nov 23 23:15 ..
```

O diretório estava vazio (`total 0`), mas a linha `.` permitiu verificar o proprietário `nwolf` e o grupo `Sales`. A linha `..` refere-se ao diretório pai, `companyA`, que continuou pertencendo a `mjackson:Personnel`.

## Resumo dos comandos praticados

| Comando | Função |
| --- | --- |
| `pwd` | Exibir o diretório atual |
| `ls` | Listar arquivos e pastas |
| `cd /home/ec2-user/companyA` | Entrar no diretório do exercício |
| `ls -l` | Mostrar detalhes, permissões e proprietário dos arquivos |
| `ls -ld pasta` | Mostrar os dados da própria pasta |
| `ls -laR` | Listar detalhes, itens ocultos e toda a árvore recursivamente |
| `sudo chown -R usuario:grupo pasta` | Alterar proprietário e grupo recursivamente |
| `sudo vi arquivo` | Criar ou editar arquivo no Vim com privilégios administrativos |
| `i`, `Esc`, `:wq` | Inserir texto, voltar ao modo normal, salvar e sair do Vim |
| `sudo chmod g+w arquivo` | Adicionar escrita ao grupo |
| `sudo chmod 764 arquivo` | Definir permissões `rwxrw-r--` |

## O que aprendi

- `chown` altera **quem é proprietário** e a qual **grupo** o arquivo pertence; `chmod` altera **quais operações são permitidas**.
- No modo simbólico, `g+w` significa adicionar gravação para o grupo.
- No modo absoluto, `r = 4`, `w = 2` e `x = 1`. Por isso, `764 = rwxrw-r--`.
- O caminho e o diretório atual precisam estar corretos antes de executar comandos recursivos.
- O Linux exige `:` para separar usuário e grupo em `chown usuario:grupo`.
- Pastas vazias também possuem proprietários e permissões: `ls -ld` ou `ls -laR` permitem conferi-los.

## Resultado

**As quatro tarefas práticas foram concluídas e validadas pelas saídas compartilhadas.** Os diretórios `HR`, `HR/Finance`, `Shipping` e `Sales` apresentaram os proprietários/grupos esperados, e os arquivos de teste mostraram as permissões desejadas.

**Observação:** a confirmação de encerramento da sessão no Vocareum não foi apresentada no histórico.

## Segurança

Em servidores reais, confirme o caminho antes de utilizar `chown -R`; evite permissões excessivas como `chmod 777`; e use `sudo` somente quando necessário. Esta documentação registra a atividade didática, não uma recomendação de configuração para produção.
