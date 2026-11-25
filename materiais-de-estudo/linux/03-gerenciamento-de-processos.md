# Guia de estudo — Gerenciamento de processos no Linux

**Escola da Nuvem | Princípios básicos do Linux**  
Resumo autoral dos slides e anotações sobre ps, pstree, top, kill, nice, jobs, at e cron.

## 1. Programa e processo
- **Programa:** conjunto de instruções armazenadas no computador.
- **Processo:** programa que está em execução.
- **PID:** identificador numérico atribuído a um processo.
- **Processo filho:** criado por outro processo, denominado processo pai.
- **PATH:** variável de ambiente que informa onde o shell procura programas executáveis.

~~~bash
echo "$PATH"
ps
pstree -p
~~~

## 2. Estados de processos
Modelo da aula: criado → pronto → em execução → aguardando → encerrado.

| Estado no Linux | Significado |
| --- | --- |
| R (running/runnable) | Executando ou pronto para executar na CPU |
| S (sleeping) | Aguardando um evento |
| D (uninterruptible sleep) | Esperando, frequentemente por entrada/saída |
| T (stopped) | Suspenso ou sendo depurado |
| Z (zombie) | Filho já encerrado, aguardando recolhimento do estado pelo pai |

Um zumbi não é simplesmente um processo cujo pai terminou: ele próprio já encerrou, mas seu pai ainda não coletou seu status.

## 3. Consultar processos
### ps — listagem
~~~bash
ps
ps -e
ps -ef
ps aux
ps -ef | grep httpd
ps -fp 1234
~~~

**ps** consulta processos, IDs, usuários, terminais e comandos. Sem opções, costuma mostrar os processos ligados à sessão atual. Formas com e sem hífen podem usar convenções diferentes.

### pidof — identificar PID pelo nome
~~~bash
pidof sshd
~~~

### pstree — visualizar a hierarquia
~~~bash
pstree
pstree -p
pstree -a
pstree -c
~~~

Opções principais: -p exibe PID, -a exibe argumentos, -c expande ramificações semelhantes. No Debian, pstree é distribuído pelo pacote psmisc, instalável com:

~~~bash
sudo apt install psmisc
~~~

## 4. top — monitoramento em tempo real
~~~bash
top
top -b -n 1
top -hv
~~~

**Pressione q para sair.**

| Campo | O que representa |
| --- | --- |
| Tasks | Contagem de processos e estados |
| load average | Média da demanda por execução e espera ininterruptível nos últimos 1, 5 e 15 minutos |
| PID | Identificador do processo |
| %CPU e %MEM | Uso de processador e memória do processo |
| us | CPU com código de usuário |
| sy | CPU com código do kernel |
| id | CPU ociosa |
| wa | Espera de entrada/saída |
| hi / si | Interrupções de hardware / software |
| st | Tempo de CPU cedido ao hipervisor |
| Mem / Swap | RAM física e memória de troca |

**Exemplo interpretativo:** quanto maior o campo id, mais ociosa a CPU ficou no intervalo observado. A memória de buffer/cache normalmente é usada pelo Linux para otimizar o sistema.

## 5. kill — enviar sinais a processos
~~~bash
kill -15 1234
kill -9 1234
kill -19 1234
kill -CONT 1234
~~~

| Sinal | Função |
| --- | --- |
| SIGTERM (15) | Solicita encerramento normal |
| SIGKILL (9) | Finaliza forçadamente, sem permitir limpeza |
| SIGSTOP (19 em Linux) | Suspende um processo |
| SIGCONT | Retoma processo suspenso |

**Boa prática:** tente SIGTERM antes de SIGKILL. O encerramento forçado pode interromper serviços e perder dados.

## 6. nice e renice — prioridade de CPU
~~~bash
nice -n 10 comando
renice -n 10 -p 1234
~~~

**nice** inicia um programa com prioridade relativa; **renice** muda a prioridade de um processo em execução. O intervalo vai de **-20** (mais favorável ao agendamento) até **19** (menos favorável). Pode ser necessário sudo para elevar prioridade.

**Correção do material:** nice não escolhe horário de execução; quem agenda horários são at e cron.

## 7. jobs, fg, bg — controle de trabalhos
~~~bash
sleep 300 &
jobs
fg %1
# Ctrl+Z suspende processo em primeiro plano
bg %1
~~~

- **&** executa um processo em segundo plano.
- **jobs** lista trabalhos do shell atual, não todos os processos do sistema.
- **fg** retorna um trabalho ao primeiro plano.
- **Ctrl+Z** suspende um trabalho em primeiro plano.
- **bg** retoma um trabalho suspenso em segundo plano.

## 8. at e cron — agendar tarefas
### at executa uma única vez
~~~bash
echo "date >> /tmp/data.txt" | at 16:00
at -l
atrm 1
~~~

### cron executa periodicamente
~~~bash
crontab -l
crontab -e
~~~

Uma entrada típica no crontab do usuário tem seis campos:

~~~text
MIN HORA DIA_DO_MES MES DIA_DA_SEMANA COMANDO
~~~

Exemplo para executar a cada hora, no minuto zero:

~~~cron
0 * * * * /usr/bin/date >> /tmp/execucoes.log
~~~

Exemplo para executar às 16h de segunda-feira:

~~~cron
0 16 * * 1 /home/ec2-user/scripts/backup.sh
~~~

O crontab do root e de um usuário comum são distintos. Execute sudo crontab -e apenas quando houver necessidade de agendar como root. O comando para edição é crontab -e; a grafia crontab -a dos slides antigos não corresponde ao uso padrão atual.

**Cuidados:** usar caminhos absolutos, conferir PATH e permissões, verificar logs após a execução. Uma entrada aparecer em crontab -l não comprova que executou.

## 9. Diferenças importantes para prova
| Comandos | Diferença |
| --- | --- |
| ps × pstree | Lista processos × mostra hierarquia |
| ps × top | Visão pontual × monitoramento em tempo real |
| nice × renice | Prioridade ao iniciar × prioridade de processo existente |
| kill -15 × kill -9 | Encerramento solicitado × forçado |
| at × cron | Tarefa única × repetitiva |
| jobs × ps | Trabalhos do shell × processos do sistema |

## 10. Perguntas de revisão
1. O que significa PID?
2. Qual comando apresenta processos em árvore?
3. O que indica id no top?
4. Qual sinal deve ser tentado antes de SIGKILL?
5. Qual a diferença entre at e cron?
6. Como listar tarefas agendadas?

**Respostas:** 1) Identificador de processo. 2) pstree. 3) Tempo de CPU ociosa. 4) SIGTERM. 5) Única execução versus execução periódica. 6) crontab -l.

## Laboratório relacionado
[Lab 239 — Gerenciar processos](../../laboratorios/239-gerenciar-processos-linux/README.md).