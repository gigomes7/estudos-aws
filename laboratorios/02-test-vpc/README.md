# Laboratório 2 — Test VPC manual e EC2

## Objetivo

Montar uma VPC manualmente, configurar as camadas de rede e segurança e testar o acesso de uma EC2 à internet.

Este registro foi reconstruído a partir da conversa de estudo e do resumo fornecido pelo autor. Não representa uma nova execução na AWS. Os prints originais e a saída completa do terminal não estão incluídos; região, tipo da instância e outros detalhes não confirmados foram omitidos.

## Configuração

| Recurso | Configuração |
| --- | --- |
| VPC | `Test VPC` |
| CIDR da VPC | `192.168.0.0/18` |
| Sub-rede pública | `192.168.1.0/28` |
| Endereços da sub-rede | 16 no total; 11 utilizáveis na AWS |
| Internet Gateway | Conectado à VPC |
| Tabela de rotas | Rota local e `0.0.0.0/0` para o Internet Gateway |
| Network ACL | `Public Subnet NACL` |
| Security Group | `public security group` |
| Sistema da EC2 | Amazon Linux 2023 |
| Acesso remoto | SSH com chave de laboratório |

## Passo a passo

1. Criar manualmente a VPC `Test VPC` com o CIDR `192.168.0.0/18`.
2. Criar a sub-rede pública `192.168.1.0/28` dentro da VPC.
3. Criar e conectar um Internet Gateway à VPC.
4. Configurar uma tabela de rotas com a rota local e uma rota `0.0.0.0/0` para o gateway. Associar a sub-rede a essa tabela.
5. Criar e associar a `Public Subnet NACL` à sub-rede. O histórico registra a regra 100 permitindo tráfego de entrada e saída; os campos completos da regra não estão disponíveis neste registro.
6. Criar o `public security group` na VPC. O histórico registra entrada TCP nas portas 22, 80 e 443, com origem `0.0.0.0/0`, e saída de todo o tráfego para `0.0.0.0/0`.
7. Iniciar uma EC2 com Amazon Linux 2023 nessa sub-rede, associando o Security Group e a chave de laboratório. Garantir um IPv4 público para o caminho direto pelo Internet Gateway.
8. Conectar por SSH usando a chave privada guardada apenas no computador local.
9. Na EC2, executar `ping -c 4 google.com` para repetir um teste de quatro pacotes.

Exemplo de conexão, com valores que precisam ser substituídos localmente:

```bash
chmod 400 /caminho/local/chave-do-laboratorio.pem
ssh -i /caminho/local/chave-do-laboratorio.pem ec2-user@IP_PUBLICO_DA_EC2
ping -c 4 google.com
```

O comando `chmod` se aplica a Linux, macOS ou WSL. No Windows, restringir as permissões do arquivo ao usuário que fará a conexão. Nenhuma chave privada acompanha este material.

## Resultados registrados

- Acesso SSH à EC2 realizado com a chave do laboratório.
- Teste de `ping google.com`: **4 pacotes enviados, 4 recebidos e 0% de perda**.
- O resultado indica resolução de nome e comunicação ICMP de ida e volta naquele momento. Não comprova funcionamento de serviços HTTP ou HTTPS.
- A sub-rede `/28` possui `2^(32-28) = 16` endereços. Com cinco reservados pela AWS, ficam 11 utilizáveis: `192.168.1.4` a `192.168.1.14`.

## O que aprendi

- O caminho para a internet depende da combinação de IP público, rota, Internet Gateway e controles de segurança.
- A NACL atua na sub-rede e é stateless: entrada e saída precisam permitir o tráfego necessário, inclusive o retorno.
- O Security Group atua nas interfaces dos recursos e é stateful: o retorno de uma conexão permitida é liberado automaticamente.
- Uma regra de saída no Security Group permite iniciar o ping; não é preciso abrir entrada ICMP só para receber essas respostas.
- Uma sub-rede `/28` tem menos capacidade que uma `/26`.

## Cuidados de segurança

- O SSH aberto para `0.0.0.0/0` foi uma configuração registrada no laboratório. Para repetir com mais segurança, restringir a porta 22 ao IP autorizado com `/32` e remover a regra depois do uso.
- Abrir 80 e 443 apenas quando houver um serviço que precise delas. Ter a porta liberada não instala um servidor web.
- Regras amplas de NACL devem ser revistas conforme a necessidade do ambiente.
- Nunca enviar arquivos `.pem`, `.ppk`, chaves de acesso AWS, senhas ou tokens ao GitHub. Revisar prints antes de publicar.
- Guardar a chave privada com acesso restrito e não reutilizar a chave de laboratório em outros ambientes.

## Limpeza

Encerrar a EC2 quando não for mais necessária e conferir volumes, snapshots e endereços públicos que possam continuar gerando custos. Depois, remover dependências da rede, desconectar o Internet Gateway e excluir os recursos do laboratório. Excluir o registro do par de chaves na AWS não apaga cópias locais da chave privada.

## Diferença para o laboratório 1

Aqui a sub-rede é **`192.168.1.0/28`**, com **11 IPv4 utilizáveis**. No [laboratório 1](../01-first-vpc/README.md), ela é `/26`, com 59 utilizáveis. As VPCs são exercícios separados; seus blocos iguais se sobrepõem e não devem ser tratados como redes diretamente interligáveis.
