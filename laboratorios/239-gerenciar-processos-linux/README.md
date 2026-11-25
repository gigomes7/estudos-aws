# 239--Lab — Gerenciar processos no Linux

## Objetivo

Praticar o gerenciamento e a observação de processos em uma instância Linux na AWS, criar um **arquivo de log de processos**, consultar o desempenho do sistema com **`top`** e configurar uma **tarefa recorrente com Cron**.

**Data da prática:** 24/11/2026  
**Fonte:** roteiro do **239--Lab - Gerenciar processos**, Escola da Nuvem / Vocareum, e comandos e resultados compartilhados durante a execução.

## Ambiente utilizado

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Região AWS | Oregon (`us-west-2`) |
| Ambiente remoto | Instância Amazon EC2 Linux temporária |
| Acesso | SSH pelo Debian WSL no Windows |
| Usuário SSH | `ec2-user` |
| Diretório do laboratório | `/home/ec2-user/companyA` |
| Ferramentas | `pwd`, `ps`, `grep`, `tee`, `cat`, `top`, `vi`, `crontab` |

> Por segurança, o endereço IP temporário e a chave privada SSH foram omitidos.

## Tarefa 1 — Conectar à instância EC2 por SSH

A conexão foi realizada a partir do Debian WSL usando uma chave PEM do Vocareum.

Na primeira tentativa, o SSH recusou a chave porque ela estava com permissão `0755`, considerada permissiva demais:

```text
WARNING: UNPROTECTED PRIVATE KEY FILE!
Permissions 0755 ... are too open.
This private key will be ignored.
```

A permissão foi corrigida:

```bash
chmod 400 ~/labsuser239.pem
```

Uma nova tentativa de SSH foi **bem-sucedida**, confirmada pelo prompt da instância `[ec2-user@... ~]$`.

## Tarefa 2 — Criar uma lista de processos

### 1. Acessar o diretório do exercício

```bash
cd /home/ec2-user/companyA
pwd
```

**Saída confirmada:**

```text
/home/ec2-user/companyA
```

### 2. Listar processos e registrar em processes.csv

O roteiro do Vocareum solicitou:

```bash
sudo ps -aux | grep -v root | sudo tee SharedFolders/processes.csv
```

**Como o comando funciona:**

- `ps -aux`: mostra informações de processos em execução.
- `|`: envia a saída do comando anterior para o seguinte.
- `grep -v root`: exclui as linhas contendo `root`.
- `sudo tee SharedFolders/processes.csv`: mostra o resultado no terminal e grava o arquivo.

A saída real continha processos como `lsmd`, `dbus-daemon`, `rpcbind`, `chronyd`, `rngd`, `postfix`, `sshd` e `bash` sob usuários não root.

### 3. Validar o log

```bash
cat SharedFolders/processes.csv
```

O conteúdo mostrado por `cat` correspondeu à saída registrada pelo `tee`.

**Trecho observado:**

```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
libstor+  1723  0.0  0.1  12628  1856 ?        Ss   01:22   0:00 /usr/bin/lsmd -d
dbus      1724  0.0  0.4  58352  4292 ?        Ss   01:22   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
rpc       1727  0.0  0.3  67356  3340 ?        Ss   01:22   0:00 /sbin/rpcbind -w
chrony    1732  0.0  0.3 120184  3076 ?        S    01:22   0:00 /usr/sbin/chronyd -F 2
ec2-user  2959  0.0  0.4 150748  4316 ?        S    01:25   0:00 sshd: ec2-user@pts/0
ec2-user  2960  0.0  0.4 124980  4148 pts/0    Ss   01:25   0:00 -bash
```

**Nota sobre o roteiro:** o objetivo menciona omitir entradas com colchetes `[` e `]`, mas o comando fornecido pelo próprio roteiro só aplica explicitamente `grep -v root`. A saída compartilhada não mostrou entradas com colchetes. O procedimento seguido foi o comando exatamente indicado no laboratório.

## Tarefa 3 — Monitorar processos usando top

### 4. Verificar o estado do sistema

```bash
top
```

O programa abriu a visualização interativa do uso de CPU, RAM e processos.

**Dados observados na captura de tela:**

```text
top - 01:29:17 up 7 min, 1 user, load average: 0.00, 0.02, 0.00
Tasks: 88 total, 1 running, 48 sleeping, 0 stopped, 0 zombie
%Cpu(s): 0.2 us, 0.0 sy, 0.0 ni, 99.8 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
KiB Mem : 962708 total, 360332 free, 71364 used, 531012 buff/cache
KiB Swap: 0 total, 0 free, 0 used
```

**Resposta à pergunta do laboratório:** havia **1 tarefa em execução (`running`)** no momento da captura.

O valor `99.8 id` indicava CPU quase toda ociosa no instante observado.

### 5. Sair e verificar versão

Depois de sair de `top` com a tecla `q`:

```bash
top -hv
```

**Saída real:**

```text
procps-ng version 3.3.10
Usage:
  top -hv | -bcHiOSs -d secs -n max -u|U user -p pid(s) -o field -w [cols]
```

## Tarefa 4 — Criar uma tarefa agendada com Cron

### 6. Abrir o editor do Crontab de root

Foi confirmado o diretório:

```bash
pwd
```

Resultado: `/home/ec2-user/companyA`.

Em seguida:

```bash
sudo crontab -e
```

O editor Vim abriu. Foram inseridas quatro linhas, conforme o roteiro do laboratório:

```cron
SHELL=/bin/bash
PATH=/usr/bin:/bin:/usr/local/bin
MAILTO=root
0 * * * * ls -la $(find .) | sed -e 's/..csv/#####.csv/g' > /home/ec2-user/companyA/SharedFolders/filteredAudit.csv
```

No Vim, foi usado `i` para editar e `Esc`, seguido de `:wq`, para salvar e sair.

**O que a configuração representa:**

- `SHELL=/bin/bash`: solicita a execução dos comandos no Bash.
- `PATH=...`: configura os diretórios usados para localizar executáveis.
- `MAILTO=root`: direciona notificações do Cron para root.
- `0 * * * *`: executa no minuto zero de **todas as horas**.
- O comando com `ls`, `find`, `sed` e `>` direciona a saída ao arquivo `filteredAudit.csv`.

### 7. Validar o agendamento

```bash
sudo crontab -l
```

**Mensagens e saída reais compartilhadas:**

```text
no crontab for root - using an empty one
crontab: installing new crontab
```

O comando `sudo crontab -l` exibiu as quatro linhas exatamente como foram configuradas:

```cron
SHELL=/bin/bash
PATH=/usr/bin:/bin:/usr/local/bin
MAILTO=root
0 * * * * ls -la $(find .) | sed -e 's/..csv/#####.csv/g' > /home/ec2-user/companyA/SharedFolders/filteredAudit.csv
```

**Resultado validado:** a tarefa foi instalada no Crontab de `root`.

**Limitação da evidência:** não foi apresentada uma execução automática do Cron, nem conferida a criação do `filteredAudit.csv`. Além disso, o comando usa `find .` com caminho relativo: o diretório de execução do Cron pode não ser o `companyA` onde o usuário trabalhou. Esta documentação preserva o comando original do roteiro e não afirma que a auditoria tenha sido executada com sucesso.

## Resumo dos comandos usados

| Comando | Para que serve |
| --- | --- |
| `chmod 400 chave.pem` | Restringir a leitura da chave SSH ao proprietário |
| `ssh -i chave.pem usuario@servidor` | Autenticar-se em um servidor Linux via SSH |
| `pwd` | Mostrar o diretório atual |
| `cd /home/ec2-user/companyA` | Entrar no diretório do laboratório |
| `ps -aux` | Exibir processos e informações de recursos |
| `grep -v root` | Excluir da saída as linhas que contêm `root` |
| `tee arquivo` | Mostrar a saída e salvá-la em arquivo |
| `cat arquivo` | Ler o conteúdo de um arquivo |
| `top` | Monitorar processos, CPU e memória em tempo real |
| `q` dentro do top | Fechar a visualização interativa |
| `top -hv` | Exibir informações de ajuda/versão |
| `sudo crontab -e` | Editar os agendamentos do usuário root |
| `i`, `Esc`, `:wq` no Vim | Inserir texto, sair do modo de inserção, salvar e sair |
| `sudo crontab -l` | Listar os agendamentos de root |

## O que aprendi

1. Em servidores Linux, a chave SSH precisa estar protegida com permissões adequadas para permitir a autenticação.
2. O comando `ps` permite consultar os processos, enquanto `grep` filtra os resultados e `tee` grava um log.
3. O comando `top` ajuda a entender o estado dos processos e a utilização de recursos.
4. O Cron permite agendar tarefas recorrentes; `0 * * * *` representa uma execução por hora.
5. `crontab -l` comprova que um agendamento foi instalado, mas não comprova que ele executou ou gerou o arquivo esperado.

## Resultado e observações

**Todas as etapas práticas até a validação do Crontab foram realizadas e conferidas com os registros do terminal.** O arquivo `SharedFolders/processes.csv` foi criado e lido, o `top` foi analisado e a tarefa recorrente foi instalada.

**Não confirmado no histórico:** execução efetiva da auditoria pelo Cron, criação do `filteredAudit.csv` e encerramento final do ambiente no Vocareum (etapas 37–38).

## Segurança

Não publicar chaves PEM, senhas, tokens ou IPs temporários desnecessários. Em ambientes de produção, validar caminhos, permissões e impacto antes de agendar rotinas com `sudo` ou como `root`. Este README documenta uma execução didática e não representa nova execução na AWS durante a publicação.
