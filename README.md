# Estudos AWS

Meus registros de estudo e laboratórios práticos na AWS, com explicações simples, passo a passo, resultados e aprendizados.

## Laboratórios

| Laboratório | Conteúdo | Sub-rede |
| --- | --- | --- |
| [01 — First VPC](laboratorios/01-first-vpc/README.md) | VPC, sub-rede pública, Internet Gateway e roteamento | `192.168.1.0/26`: 59 IPv4 utilizáveis |
| [02 — Test VPC manual e EC2](laboratorios/02-test-vpc/README.md) | VPC manual, rotas, NACL, Security Group, Amazon Linux 2023, SSH e ping | `192.168.1.0/28`: 11 IPv4 utilizáveis |

As duas VPCs usam `192.168.0.0/18`, em exercícios separados. No segundo laboratório, o teste de conexão registrou 4 pacotes enviados, 4 recebidos e 0% de perda.

## Sobre os registros

Os textos foram organizados a partir do histórico das aulas e do resumo dos laboratórios. Não indicam uma nova execução na AWS. Detalhes não confirmados e capturas originais não foram adicionados.

## Segurança

- Não publicar chaves privadas, credenciais, senhas, tokens nem dados confidenciais.
- Restringir SSH ao IP autorizado e liberar somente o tráfego necessário.
- Revisar imagens e saídas de terminal antes de compartilhá-las.
- Remover recursos de laboratório após o uso e conferir possíveis cobranças.

Este repositório contém documentação de estudo; as configurações de laboratório precisam ser revistas antes de qualquer uso em produção.
