# Laboratório 1 — First VPC

## Objetivo

Criar uma rede na AWS e entender como a VPC, a sub-rede pública, o Internet Gateway e a tabela de rotas trabalham juntos.

Este registro foi reconstruído a partir da conversa de estudo. Não representa uma nova execução na AWS. Detalhes não informados, como região e zona de disponibilidade, foram omitidos.

## Configuração

| Recurso | Configuração |
| --- | --- |
| VPC | `First VPC` |
| CIDR da VPC | `192.168.0.0/18` |
| Sub-rede pública | `192.168.1.0/26` |
| Endereços da sub-rede | 64 no total; 59 utilizáveis na AWS |
| Acesso à internet | Internet Gateway conectado à VPC e rota na tabela da sub-rede |

## Passo a passo

1. No console de VPC, criar a VPC `First VPC` com o bloco IPv4 `192.168.0.0/18`.
2. Criar uma sub-rede dentro dessa VPC com o bloco `192.168.1.0/26`.
3. Criar um Internet Gateway e conectá-lo à `First VPC`.
4. Na tabela de rotas usada pela sub-rede, manter a rota local da VPC e adicionar `0.0.0.0/0` com destino ao Internet Gateway.
5. Conferir a associação da sub-rede com essa tabela e o estado do gateway.

A rota para o Internet Gateway torna a sub-rede pública. Para uma instância acessar a internet por IPv4, ela também precisa de IPv4 público e regras de segurança compatíveis.

## Resultados registrados

- VPC e sub-rede pública configuradas com os blocos acima.
- Internet Gateway e roteamento configurados para o caminho de acesso à internet.
- Sub-rede `/26` com `2^(32-26) = 64` endereços. A AWS reserva cinco, deixando 59 utilizáveis: `192.168.1.4` a `192.168.1.62`.
- Não há teste de EC2, SSH ou ping documentado para este laboratório neste registro.

## O que aprendi

- A VPC é a rede principal; a sub-rede divide essa rede em uma parte menor.
- O CIDR define o tamanho do bloco de IPs. Quanto maior o número após a barra, menor o bloco.
- O Internet Gateway fornece o caminho para a internet, e a tabela de rotas indica quando usá-lo.
- Criar uma sub-rede, por si só, não libera acesso à internet.

## Cuidados de segurança e limpeza

- Liberar somente o tráfego necessário nos controles de rede.
- Nunca publicar chaves privadas, credenciais, IDs de conta ou imagens sem revisar dados sensíveis.
- Ao terminar, remover os recursos de laboratório que não forem mais necessários. Antes de excluir a VPC, remover dependências e desconectar o Internet Gateway.
- Conferir no faturamento se restaram recursos cobrados.

## Diferença para o laboratório 2

Aqui a sub-rede é **`192.168.1.0/26`**, com **59 IPv4 utilizáveis**. No [laboratório 2](../02-test-vpc/README.md), ela é `/28`.
