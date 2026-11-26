# 245 — Gerenciar arquivos de log no Linux

**Data:** 26/11/2026 | **Ambiente:** instância EC2 temporária do Vocareum em us-west-2, acesso SSH a partir do Debian WSL.

## Objetivo

Examinar um arquivo de log de autenticação com o comando less e identificar a última entrada de cada usuário usando lastlog.

## Tarefa 1 — Acessar a EC2 via SSH

A conexão ao servidor do laboratório com o usuário ec2-user foi confirmada.

**Boas práticas:** a chave privada PEM e os IPs da sessão não são publicados. A chave deve ter permissões restritas, por exemplo:

~~~bash
chmod 400 labsuser.pem
ssh -i labsuser.pem ec2-user@<IP_PUBLICO_DO_LAB>
~~~

Os comandos acima são referência de acesso, não uma transcrição literal dos nomes de arquivos utilizados.

## Tarefa 2 — Revisar logs de segurança

### Etapa 21 — Validar pasta de trabalho

~~~bash
pwd
~~~

A usuária confirmou estar na pasta companyA. Caminho esperado no roteiro:

~~~text
/home/ec2-user/companyA
~~~

Caso necessário, usa-se cd companyA a partir do diretório pessoal.

### Etapas 22 e 23 — Ler o arquivo secure

~~~bash
sudo less /tmp/log/secure
~~~

**Resultado confirmado:** o arquivo foi aberto e exibiu registros de SSH, PAM e FTP, inclusive mensagens semelhantes às seguintes (IPs suprimidos):

~~~text
Failed password for root from [IP_OCULTO] port [PORTA] ssh2
Disconnecting: Too many authentication failures for root [preauth]
pam_unix(sudo:session): session opened for user root by (uid=0)
pam_unix(sudo:session): session closed for user root
START: ftp pid=[PID] from=[IP_OCULTO]
error: Bind to port 22 on 0.0.0.0 failed: Address already in use.
~~~

**Leitura dos eventos:**
- Failed password: autenticação malsucedida.
- Too many authentication failures: a sessão foi encerrada após muitas falhas.
- pam_unix(sudo:session): abertura/encerramento de sessão associada ao sudo.
- START: ftp: início de conexão FTP.
- Address already in use: tentativa de vincular serviço a uma porta já em uso.

Amostra fornecida pelo laboratório: o arquivo /tmp/log/secure não deve ser confundido com os eventos atuais da EC2. Em sistemas como Amazon Linux 2 e CentOS 7, o caminho tradicional do log de autenticação é /var/log/secure. A saída analisada continha registros identificados como centos7.

A usuária pressionou q, saiu do less e retornou ao prompt com sucesso.

### Etapa 24 — Consultar último login dos usuários

~~~bash
sudo lastlog
~~~

**Resultado real resumido (IP de origem omitido):**

~~~text
Username      Port   From          Latest
root                               **Never logged in**
bin                                **Never logged in**
daemon                             **Never logged in**
ec2-user      pts/0  [IP_OCULTO]   Thu Nov 26 23:26:01 +0000 2026
ljuan                              **Never logged in**
mmajor                             **Never logged in**
mjackson                            **Never logged in**
eowusu                              **Never logged in**
nwolf                               **Never logged in**
~~~

O comando executou sem erro. O usuário ec2-user tinha um login registrado em 26/11/2026 às 23:26:01 UTC (20:26:01 em Brasília). Para os demais usuários exibidos com Never logged in, não existia registro de login anterior no histórico consultado pelo lastlog.

**Atenção para o quiz:**
- lastlog: mostra o último login registrado de cada usuário.
- last: lista registros históricos de sessões/login, quando disponíveis.
- users: lista nomes de usuários conectados atualmente.

## Referência rápida

| Comando ou caminho | Finalidade |
| --- | --- |
| pwd | Mostrar o diretório atual |
| cd companyA | Acessar pasta de trabalho |
| /tmp/log/secure | Arquivo de exemplo usado pelo laboratório |
| /var/log/secure | Caminho tradicional de logs de autenticação nesta família Linux |
| sudo less /tmp/log/secure | Examinar logs paginados |
| q | Sair do visualizador less |
| sudo lastlog | Consultar último login por usuário |
| last | Examinar histórico de sessões |

## Resultado

**Confirmado:** acesso à EC2, diretório companyA validado, visualização dos logs de exemplo, saída do less e execução bem-sucedida de sudo lastlog.

**Encerramento do Vocareum:** o comando de finalização foi orientado no chat; a confirmação do clique em Encerrar laboratório não foi observada diretamente.

**Segurança:** não foram incluídos chaves privadas, nomes de arquivo reais da chave, IP público da sessão, IP de origem do login nem credenciais de acesso.
