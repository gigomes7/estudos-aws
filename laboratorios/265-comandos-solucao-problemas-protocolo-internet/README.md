# 265--Lab - Comandos de solução de problemas de protocolo da Internet

## Objetivo

Praticar o diagnóstico de problemas de rede a partir de uma instância Amazon EC2 usando comandos Linux. A ideia é conferir se o servidor consegue alcançar outros endereços, enxergar o caminho dos pacotes, identificar conexões TCP e testar serviços HTTP/HTTPS.

Este registro foi montado a partir do roteiro do laboratório e dos resultados executados e compartilhados em **10/11/2026**. A execução ocorreu em um ambiente temporário do Vocareum.

## Ambiente utilizado

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Região | Oregon (`us-west-2`) |
| Instância fornecida pelo laboratório | Amazon EC2 com **Amazon Linux 2** |
| Acesso | SSH a partir do Debian no WSL (Windows) |
| Usuário remoto | `ec2-user` |
| Testes | `ping`, `traceroute`, `netstat`, `telnet`, `curl` |

> O IP público e a chave SSH foram temporários e não estão publicados aqui. O laboratório já disponibilizava a EC2; não foi necessário criar uma instância.

## Passo a passo e resultados

### 1. Acessar a EC2 por SSH

Baixar a chave `.pem` do laboratório atual no Vocareum, copiá-la para o diretório pessoal do Linux/WSL e ajustar as permissões:

```bash
chmod 400 ~/chave-do-laboratorio.pem
ssh -i ~/chave-do-laboratorio.pem ec2-user@IP_PUBLICO_DA_EC2
```

**Resultado:** acesso remoto realizado com sucesso. O prompt da instância indicou o usuário `ec2-user` em um servidor Amazon Linux 2.

> Nunca colocar a chave privada `.pem` no repositório.

### 2. `ping` — testar conectividade IP (camada 3 do modelo OSI)

```bash
ping 8.8.8.8 -c 5
```

**Resultado observado:**

```text
5 packets transmitted, 5 received, 0% packet loss
rtt min/avg/max/mdev = 6.084/6.138/6.316/0.089 ms
```

**O que significa:** a EC2 enviou cinco solicitações ICMP ao endereço `8.8.8.8` e recebeu cinco respostas. Para esse destino e naquele momento, a conectividade IP funcionou sem perda de pacotes. Como o destino era um IP, este teste **não comprovou resolução DNS**.

### 3. `traceroute` — investigar o caminho do pacote (camada 3)

```bash
traceroute 8.8.8.8
```

**Resultado observado:** o destino `8.8.8.8` foi alcançado no **5º salto**. O 4º salto mostrou `* * *`, mas o salto seguinte respondeu.

**O que significa:** o traceroute revelou parte do caminho até o destino. Um salto com asteriscos pode simplesmente não responder às sondas; como o destino final respondeu, isso não comprova uma falha naquele ponto.

### 4. `netstat` — consultar conexões TCP (camada 4)

```bash
netstat -tp
```

**Resultado observado:** conexão SSH na porta 22 com estado `ESTABLISHED`. O comando também apresentou um aviso de que não conseguiu mostrar alguns nomes de processo sem permissões de administrador.

**O que significa:** havia uma conexão TCP ativa entre o computador local e a EC2. O aviso de permissão não indica que a conexão falhou.

### 5. `telnet` — testar uma porta TCP (camada 4)

O ambiente Amazon Linux 2 exigiu a instalação da ferramenta:

```bash
sudo yum install telnet -y
telnet www.google.com 80
```

**Resultado observado:**

```text
Connected to www.google.com.
```

**O que significa:** a EC2 conseguiu abrir uma conexão TCP com a **porta 80 (HTTP)** do destino. Isso não verifica, por si só, o conteúdo de uma página web.

> O Telnet é usado aqui apenas como ferramenta de teste de porta. Ele não deve ser escolhido para administração remota segura, pois não criptografa a sessão.

### 6. `curl` — testar HTTPS e resposta web (camada 7)

```bash
curl -vLo /dev/null https://aws.com
```

- `-v`: mostra detalhes da conexão.
- `-L`: acompanha redirecionamentos.
- `-o /dev/null`: descarta o corpo da resposta.

**Resultados observados:**

| Endereço | Resposta |
| --- | --- |
| `https://aws.com` | `HTTP/2 302` — redirecionamento |
| `https://www.aws.com` | `HTTP/2 301` — redirecionamento |
| `https://aws.amazon.com` | `HTTP/2 200` — resposta bem-sucedida |

O teste também registrou a conexão na porta **443**, negociação **TLS 1.2** e a mensagem `SSL certificate verify ok`.

**O que significa:** a EC2 conseguiu resolver os nomes, criar conexões HTTPS, validar os certificados apresentados e receber uma resposta HTTP de sucesso do destino final.

## Resumo dos testes

| Ferramenta | O que foi conferido | Resultado |
| --- | --- | --- |
| SSH | Acesso à instância | Conectou |
| `ping` | ICMP para `8.8.8.8` | 5/5 respostas; 0% de perda |
| `traceroute` | Caminho até `8.8.8.8` | Destino alcançado no 5º salto |
| `netstat` | Conexões TCP | SSH `ESTABLISHED` |
| `telnet` | TCP na porta 80 | `Connected` |
| `curl` | HTTPS e resposta de aplicação | Redirecionamentos 302/301 e `200 OK` |

## O que aprendi

- `ping` ajuda a saber se um endereço responde por ICMP, mas não comprova que um site ou uma porta estejam disponíveis.
- `traceroute` ajuda a investigar por onde o tráfego passa; um `* * *` isolado não significa necessariamente falha.
- `netstat` ajuda a enxergar conexões TCP e seus estados, como `ESTABLISHED`.
- `telnet` pode testar se é possível abrir uma conexão TCP com determinada porta.
- `curl` testa o comportamento de um serviço HTTP/HTTPS e mostra códigos de resposta, redirecionamentos e detalhes TLS.
- Usar os comandos em sequência evita concluir que “a internet caiu” sem identificar qual parte da comunicação falhou.

## Segurança e observações

- Não publicar arquivos `.pem`/`.ppk`, senhas, tokens, credenciais AWS ou detalhes desnecessários de contas.
- Evitar expor SSH (porta 22) para toda a internet em ambientes reais; limitar o acesso às origens autorizadas.
- Os testes mostram o funcionamento dos **destinos, portas e protocolos testados**, não de toda a conectividade de rede.
- As capturas de tela originais não foram incluídas nesta publicação; os resultados acima foram transcritos dos registros do laboratório.
- Ao concluir o exercício, encerrar o laboratório Vocareum e conferir eventuais recursos que precisem ser removidos.

## Referência

- Roteiro fornecido pela Escola da Nuvem / Vocareum: **265--Lab - Comandos de solução de problemas de protocolo da Internet**.
