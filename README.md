# Estudos AWS

Meus registros de estudo e laboratórios práticos na AWS, com explicações simples, passo a passo, resultados e aprendizados.

## Laboratórios

| Laboratório | Conteúdo | Resultado principal |
| --- | --- | --- |
| [01 — First VPC](laboratorios/01-first-vpc/README.md) | VPC, sub-rede pública, Internet Gateway e roteamento | Sub-rede `192.168.1.0/26`: 59 IPv4 utilizáveis |
| [02 — Test VPC manual e EC2](laboratorios/02-test-vpc/README.md) | VPC manual, rotas, NACL, Security Group, Amazon Linux 2023, SSH e ping | Sub-rede `192.168.1.0/28`: 11 IPv4 utilizáveis; ping 4/4 |
| [265 — Comandos de solução de problemas de protocolo da Internet](laboratorios/265-comandos-solucao-problemas-protocolo-internet/README.md) | SSH, ping, traceroute, netstat, telnet e curl em EC2 Linux | Ping 5/5, TCP/80 conectado, HTTPS `200 OK` |
| [266 — Solucionar problemas de rede](laboratorios/266-solucionar-problemas-de-rede/README.md) | Diagnóstico do Apache, Security Group, Network ACL, curl e ss | Apache iniciado, porta HTTP liberada e página de teste aberta |

As duas VPCs dos laboratórios 01 e 02 usam `192.168.0.0/18`, em exercícios separados. Seus blocos de sub-rede são diferentes: `/26` e `/28`.

## Sobre os registros

Os textos foram organizados a partir do histórico das aulas e dos resultados dos laboratórios. Os registros documentam a execução que foi relatada e não representam uma nova execução na AWS durante a publicação. Detalhes não confirmados e capturas originais não foram adicionados.

## Segurança

- Não publicar chaves privadas, credenciais, senhas, tokens nem dados confidenciais.
- Restringir SSH ao IP autorizado e liberar somente o tráfego necessário.
- Revisar imagens e saídas de terminal antes de compartilhá-las.
- Remover recursos de laboratório após o uso e conferir possíveis cobranças.

Este repositório contém documentação de estudo; as configurações de laboratório precisam ser revistas antes de qualquer uso em produção.
