# Estudos AWS

Meus registros de estudo e laboratórios práticos na AWS, com explicações simples, passo a passo, resultados e aprendizados.

## Laboratórios

| Laboratório | Conteúdo | Resultado principal |
| --- | --- | --- |
| [01 — First VPC](laboratorios/01-first-vpc/README.md) | VPC, sub-rede pública, Internet Gateway e roteamento | Sub-rede `192.168.1.0/26`: 59 IPv4 utilizáveis |
| [02 — Test VPC manual e EC2](laboratorios/02-test-vpc/README.md) | VPC manual, rotas, NACL, Security Group, Amazon Linux 2023, SSH e ping | Sub-rede `192.168.1.0/28`: 11 IPv4 utilizáveis; ping 4/4 |
| [229 — Criar usuários e grupos no Linux](laboratorios/229-criar-usuarios-grupos-linux/README.md) | Usuários, grupos, permissões, sudo e auditoria de acesso | Grupos conferidos; permissão de escrita e sudo negados conforme esperado |
| [231 — Editar arquivos com Vim e Nano](laboratorios/231-editar-arquivos-vim-nano/README.md) | vimtutor (lições 1–3), edição, salvamento e descarte de alterações | Arquivos `helloworld` e `cloudworld` editados e conferidos |
| [233 — Trabalhar com o sistema de arquivos](laboratorios/233-trabalhar-sistema-arquivos/README.md) | mkdir, touch, cp, mv, rm, rmdir e tree em EC2 Linux | Estrutura CompanyA/HR organizada com 3 subpastas e 6 arquivos |
| [235 — Trabalhar com arquivos no Linux](laboratorios/235-trabalhar-arquivos-linux/README.md) | tar, backup, registro CSV com tee e movimentação de arquivos | Backup criado e movido para IA; data registrada em backups.csv |
| [237 — Gerenciar permissões de arquivos no Linux](laboratorios/237-gerenciar-permissoes-arquivos/README.md) | chown, chmod simbólico e numérico, proprietários e grupos | HR, Finance, Shipping e Sales com propriedades validadas; chmod 764 confirmado |
| [239 — Gerenciar processos no Linux](laboratorios/239-gerenciar-processos-linux/README.md) | SSH, ps, grep, tee, top e agendamento Cron | processes.csv conferido; top com 1 tarefa running; crontab instalado |
| [241 — Gerenciar serviços no Linux](laboratorios/241-gerenciar-servicos-linux/README.md) | Apache/httpd, systemctl, top, stress.sh e CloudWatch | Apache ativo e testado via HTTP; pico de CPU de ~63,69% registrado |
| [243 — Gerenciamento de software no Linux](laboratorios/243-gerenciamento-software-linux/README.md) | YUM, histórico/undo, Python, pip, AWS CLI e EC2 | httpd instalado e revertido; consulta AWS CLI confirmou t3.micro |
| [245 — Gerenciar arquivos de log no Linux](laboratorios/245-gerenciar-arquivos-log-linux/README.md) | less, logs de autenticação, eventos SSH/PAM e lastlog | Leitura de secure confirmada; último login do ec2-user identificado |
| [265 — Comandos de solução de problemas de protocolo da Internet](laboratorios/265-comandos-solucao-problemas-protocolo-internet/README.md) | SSH, ping, traceroute, netstat, telnet e curl em EC2 Linux | Ping 5/5, TCP/80 conectado, HTTPS `200 OK` |
| [266 — Solucionar problemas de rede](laboratorios/266-solucionar-problemas-de-rede/README.md) | Diagnóstico do Apache, Security Group, Network ACL, curl e ss | Apache iniciado, porta HTTP liberada e página de teste aberta |

As duas VPCs dos laboratórios 01 e 02 usam `192.168.0.0/18`, em exercícios separados. Seus blocos de sub-rede são diferentes: `/26` e `/28`.

## Materiais de estudo — Linux

Resumos autorais organizados a partir dos slides e anotações das aulas da Escola da Nuvem, com comandos explicados, exemplos, comparações, correções de traduções confusas e perguntas de revisão.

| Aula | Principais assuntos | Laboratório relacionado |
| --- | --- | --- |
| [01 — Trabalhar com arquivos](materiais-de-estudo/linux/01-trabalhar-com-arquivos.md) | find, grep, diff, links, inodes, tar e compressão | 235 |
| [02 — Permissões de arquivos](materiais-de-estudo/linux/02-permissoes-de-arquivos.md) | ls -l, rwx, chmod simbólico e octal, chown, sudo | 237 |
| [03 — Gerenciamento de processos](materiais-de-estudo/linux/03-gerenciamento-de-processos.md) | ps, pstree, top, kill, nice, jobs, at e cron | 239 |
| [04 — Gerenciamento de software](materiais-de-estudo/linux/04-gerenciamento-de-software.md) | RPM, YUM, DPKG, APT, repositórios, wget, curl e AWS CLI | 243 |
| [05 — Gerenciamento de arquivos de log](materiais-de-estudo/linux/05-gerenciamento-de-arquivos-de-log.md) | Syslog, /var/log, head, tail, less, grep, lastlog e logrotate | 245 |

**Observação:** estes materiais são notas de estudo, não cópias integrais dos slides originais, e os comandos ilustrativos não representam execução adicional na AWS.

## Sobre os registros

Os textos foram organizados a partir do histórico das aulas e dos resultados dos laboratórios. Os registros documentam a execução que foi relatada e não representam uma nova execução na AWS durante a publicação. Detalhes não confirmados e capturas originais não foram adicionados.

## Segurança

- Não publicar chaves privadas, credenciais, senhas, tokens nem dados confidenciais.
- Restringir SSH ao IP autorizado e liberar somente o tráfego necessário.
- Revisar imagens e saídas de terminal antes de compartilhá-las.
- Remover recursos de laboratório após o uso e conferir possíveis cobranças.

Este repositório contém documentação de estudo; as configurações de laboratório precisam ser revistas antes de qualquer uso em produção.
