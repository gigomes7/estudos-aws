# 233--Lab — Trabalhar com o sistema de arquivos

## Objetivo

Praticar comandos Linux de navegação e gerenciamento de arquivos em uma instância EC2: criar pastas, criar arquivos vazios, copiar, mover, excluir e validar toda a estrutura.

**Data da execução:** 18/11/2026  
**Fonte:** laboratório da Escola da Nuvem / Vocareum, com comandos e saídas do terminal compartilhados durante a prática.

## Ambiente utilizado

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Região | Oregon (`us-west-2`) |
| Servidor | Instância Amazon EC2 Linux do laboratório |
| Acesso | SSH pelo Debian no WSL (Windows) |
| Usuário remoto | `ec2-user` |
| Comandos principais | `pwd`, `mkdir`, `cd`, `ls`, `touch`, `cp`, `rm`, `rmdir`, `mv`, `tree` |

> A chave SSH e o IP público usados na sessão eram temporários e foram omitidos da documentação.

## Passo a passo e evidências

### 1. Criar a estrutura inicial

Após conectar à instância EC2:

```bash
pwd
mkdir CompanyA
cd CompanyA
mkdir Finance HR Management
ls
```

**Saída observada:**

```text
Finance  HR  Management
```

Em seguida, foram criados arquivos `.csv` vazios:

```bash
cd HR
touch Assessments.csv TrialPeriod.csv
ls

cd ../Finance
touch Salary.csv ProfitAndLossStatements.csv
ls

cd ..
touch Management/Managers.csv Management/Schedule.csv
ls Management
ls -laR
```

**Correção feita durante a prática:** o arquivo `assessments.csv` havia sido criado com inicial minúscula. Como Linux diferencia maiúsculas e minúsculas, foi renomeado:

```bash
mv HR/assessments.csv HR/Assessments.csv
ls HR
```

**Resultado:** todas as três pastas e os seis arquivos foram listados com os nomes esperados.

### 2. Copiar a pasta financeira para RH

Na pasta `CompanyA`:

```bash
cp -r Finance HR
ls HR/Finance
```

**Saída observada:**

```text
ProfitAndLossStatements.csv  Salary.csv
```

O parâmetro `-r` faz a cópia recursiva do diretório e dos arquivos contidos nele.

### 3. Testar exclusão de diretório não vazio

```bash
rmdir Finance
```

**Erro esperado e observado:**

```text
rmdir: failed to remove 'Finance': Directory not empty
```

O `rmdir` remove apenas diretórios vazios. Seguindo a ordem do laboratório, os arquivos originais foram excluídos antes da pasta:

```bash
rm Finance/ProfitAndLossStatements.csv Finance/Salary.csv
ls Finance
rmdir Finance
ls
```

**Resultado observado:**

```text
HR  Management
```

A cópia em `HR/Finance` foi mantida.

### 4. Mover a pasta Management para HR

```bash
mv Management HR
ls . HR/Management
```

**Resultado observado:** a pasta `CompanyA` passou a conter apenas `HR`, e `HR/Management` continha `Managers.csv` e `Schedule.csv`.

### 5. Criar Employees e mover os arquivos de RH

```bash
cd HR
mkdir Employees
mv Assessments.csv TrialPeriod.csv Employees
ls . Employees
```

**Saída observada:**

```text
.:
Employees  Finance  Management

Employees:
Assessments.csv  TrialPeriod.csv
```

### 6. Validar a estrutura com tree

O comando `tree` não estava instalado inicialmente (`command not found`), então foi instalado no ambiente temporário:

```bash
sudo yum install tree -y
tree ..
```

**Saída real compartilhada no laboratório:**

```text
..
└── HR
    ├── Employees
    │   ├── Assessments.csv
    │   └── TrialPeriod.csv
    ├── Finance
    │   ├── ProfitAndLossStatements.csv
    │   └── Salary.csv
    └── Management
        ├── Managers.csv
        └── Schedule.csv
```

**Resultado final confirmado:** a estrutura solicitada foi reproduzida, com três subpastas sob `HR` e seis arquivos `.csv`. A parte prática foi concluída; o encerramento da sessão no Vocareum não foi confirmado nos registros apresentados.

## Resumo dos comandos

| Comando | Função |
| --- | --- |
| `pwd` | Mostrar o caminho do diretório atual |
| `mkdir` | Criar diretórios |
| `cd` | Navegar entre diretórios |
| `ls`, `ls -laR` | Listar arquivos, detalhes e subpastas |
| `touch` | Criar arquivos vazios |
| `cp -r` | Copiar pastas e conteúdos |
| `rm` | Excluir arquivos |
| `rmdir` | Excluir diretórios vazios |
| `mv` | Mover ou renomear |
| `tree` | Mostrar a estrutura em formato de árvore |

## O que aprendi

- O Linux diferencia maiúsculas e minúsculas em nomes de arquivo.
- `..` significa diretório pai, como em `cd ..` e `tree ..`.
- `cp -r` cria uma cópia; `mv` move o item para outro local.
- `rmdir` não elimina pastas com conteúdo, evitando exclusões não intencionais.
- Validar os resultados após cada comando ajuda a identificar erros antes de prosseguir.
- Em uma EC2, os mesmos comandos podem ser usados para organizar arquivos de aplicações e ambientes de servidor.

## Segurança

Exclusões com `rm` exigem atenção: confira o caminho antes de executar. Não publicar chaves SSH, senhas ou endereços temporários desnecessários. Este README documenta a execução realizada, e não um ambiente de produção.

**Referência:** **233--Lab - Trabalhar com o sistema de arquivos**, Escola da Nuvem / Vocareum.
