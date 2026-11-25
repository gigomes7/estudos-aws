# 241 — Gerenciar serviços no Linux

**Data:** 25/11/2026 | **Ambiente:** Amazon Linux 2 em EC2, região us-west-2, acesso SSH no Debian WSL.

## Objetivo
Aprender a iniciar, inspecionar e parar o Apache HTTP Server; avaliar o desempenho no terminal usando top; gerar uma carga temporária com stress.sh; e correlacionar os dados com os gráficos do Amazon CloudWatch.

## Tarefa 1 — Conectar à EC2
A conexão SSH ao host do Vocareum foi confirmada pelo terminal remoto ec2-user.

## Tarefa 2 — Administrar o Apache
### Etapa 21: estado inicial
~~~bash
sudo systemctl status httpd.service
~~~

**Saída real:** Loaded: loaded; Active: inactive (dead). O Apache estava instalado e parado.

### Etapas 22 e 23: iniciar e confirmar
~~~bash
sudo systemctl start httpd.service
sudo systemctl status httpd.service
~~~

**Saída real:**

~~~text
Active: active (running) since Wed 2026-11-25 23:06:24 UTC
Main PID: 2671 (httpd)
~~~

### Etapa 24: teste HTTP
A página padrão **Test Page for the Apache HTTP Server 2.4** foi aberta no navegador usando o IP público temporário da EC2. A captura comprovou que o servidor respondia via HTTP.

### Etapa 25: parar
~~~bash
sudo systemctl stop httpd.service
~~~

Comando executado no terminal. Não foi apresentada uma consulta status posterior.

## Tarefa 3 — Monitoramento e teste de carga
### Etapa 26: desempenho antes do stress
~~~bash
top
~~~

A primeira captura apresentou:

| Indicador | Resultado |
| --- | --- |
| Total de tarefas | 91 |
| Running | 1 |
| Sleeping | 49 |
| Stopped / Zombie | 0 / 0 |
| RAM total | 962708 KiB |
| RAM usada | 79020 KiB |
| Load average (1 min) | 0.00 |

Foi usada a tecla q para sair do top.

### Etapas 27–28: gerar carga e comparar
O roteiro fornecia o script stress.sh, executado em segundo plano:

~~~bash
./stress.sh &
top
~~~

**Saída real:**

~~~text
stress: info: [2733] dispatching hogs: 8 cpu, 4 io, 2 vm, 0 hdd
~~~

Durante o teste, a captura do top mostrou 106 tarefas, 15 em execução, load average de 11.14, 61.7% de CPU em modo usuário, 38.3% de CPU em modo sistema e 0% ociosa. Foram observados vários processos com o nome stress.

### Etapas 29–32: CloudWatch
No console AWS, foi aberto o CloudWatch e selecionado o painel automático da EC2. O gráfico **CPU Utilization: Average** mostrou:

- utilização próxima de 0% antes da carga;
- pico aproximado de **63,69%** depois do teste;
- em captura posterior, leve redução para cerca de **57%**.

**Precisão da evidência:** a última captura mostra início de queda, mas não comprova o retorno da CPU ao patamar inicial.

Os gráficos de rede também apareceram. Alguns indicadores de disco estavam sem dados naquele intervalo de visualização.

## Referência rápida
| Comando | Função |
| --- | --- |
| systemctl status httpd.service | Verifica estado do Apache |
| systemctl start httpd.service | Inicia o servidor Apache |
| systemctl stop httpd.service | Solicita parada do servidor |
| top | Monitora CPU, RAM, load average e processos |
| q (dentro do top) | Retorna ao terminal |
| ./stress.sh & | Inicia a carga de teste em segundo plano |

## Resultado
**Confirmado:** conexão SSH, Apache ativo, página de teste HTTP acessível, comando de parada executado, observações de top antes e durante a carga e pico de CPU registrado pelo CloudWatch.

**Não comprovado:** retorno completo da CPU ao patamar inicial e confirmação do encerramento da sessão no Vocareum.

**Atenção:** scripts de estresse devem ser utilizados somente em laboratórios ou ambientes expressamente autorizados. O IP público e as chaves temporárias não foram incluídos.