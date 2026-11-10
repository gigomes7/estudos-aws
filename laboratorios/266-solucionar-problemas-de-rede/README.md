# 266--Lab - Solucionar problemas de rede

**Data:** 10/11/2026  
**Ambiente:** AWS / Vocareum — região Oregon (`us-west-2`)  
**Cenário:** a cliente Ana não conseguia acessar um servidor Apache pelo navegador.

## Objetivo

Descobrir por que o servidor web não abria pelo IP público e corrigir o que estivesse bloqueando a conexão.

A arquitetura do exercício já estava montada: uma **VPC**, **sub-rede pública**, **Internet Gateway** e **instância EC2**. O trabalho era investigar o servidor e as configurações de rede, sem sair alterando tudo de uma vez.

## Ambiente usado

| Item | Informação |
| --- | --- |
| Sistema da EC2 | Amazon Linux 2 |
| Acesso à EC2 | SSH pelo Debian no WSL (Windows) |
| Usuário do Linux | `ec2-user` |
| Serviço web | Apache HTTP Server (`httpd`) |
| Protocolo do site | HTTP, porta TCP 80 |
| IPs | Endereços temporários do laboratório, omitidos na documentação |

> Não publique a chave `.pem`: ela permite autenticação SSH e deve ficar somente no computador autorizado.

## Passo a passo: o que foi feito

### 1. Acessar a EC2 por SSH

Foi baixada a chave PEM do Vocareum. No Debian/WSL, a chave foi copiada para a pasta pessoal e teve as permissões ajustadas:

```bash
# Exemplo: substitua pelo arquivo obtido no seu próprio laboratório
chmod 400 ~/chave-lab266.pem
ssh -i ~/chave-lab266.pem ec2-user@IP_PUBLICO
```

**Resultado:** conexão SSH estabelecida com a EC2 Amazon Linux 2.

### 2. Conferir o Apache

```bash
sudo systemctl status httpd.service
```

O serviço estava instalado, mas parado:

```text
Loaded: loaded
Active: inactive (dead)
```

Isso já explicava por que o Apache não conseguiria atender uma solicitação web naquele momento.

### 3. Iniciar o Apache e conferir novamente

```bash
sudo systemctl start httpd.service
sudo systemctl status httpd.service
```

**Resultado:**

```text
Active: active (running)
```

O Apache passou a executar normalmente. Porém, **o site ainda não abria** no navegador; a tentativa de acesso HTTP pelo IP público retornou `ERR_TIMED_OUT`.

### 4. Investigar o Security Group

No console AWS, foi consultado o **Security Group associado ao servidor**.

**Antes:**

| Serviço | Porta | Origem | Situação |
| --- | --- | --- | --- |
| SSH | TCP 22 | `0.0.0.0/0` | Permitido |
| HTTP | TCP 80 | — | **Sem regra de entrada** |

O SSH funcionava, mas o grupo de segurança não permitia novas conexões HTTP externas.

**Correção realizada:** foi adicionada uma regra de entrada **HTTP / TCP 80 / `0.0.0.0/0`**, mantendo a regra SSH existente.

> O acesso HTTP aberto a qualquer origem foi usado apenas para reproduzir o exercício de servidor web público. Em produção, as permissões devem ser avaliadas conforme a aplicação. Para SSH, o mais seguro é limitar a origem aos IPs autorizados, e não `0.0.0.0/0`.

### 5. Conferir a Network ACL da sub-rede

A sub-rede pública utilizava a **Network ACL padrão da VPC**.

As regras observadas eram:

| Direção | Regra 100 | Regra `*` |
| --- | --- | --- |
| Entrada | Allow — todo o tráfego, `0.0.0.0/0` | Deny para o que não corresponder antes |
| Saída | Allow — todo o tráfego, `0.0.0.0/0` | Deny para o que não corresponder antes |

**Conclusão dessa verificação:** a ACL não estava bloqueando o tráfego de entrada ou saída necessário ao teste. Nenhuma alteração foi feita nela.

### 6. Testar o Apache dentro da própria EC2

Com o navegador ainda apresentando timeout, foi executado:

```bash
curl -I --max-time 5 http://127.0.0.1/
```

**Resultado:**

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.69
```

Isso mostrou que o Apache **respondia internamente**, mas retornava `403` para essa requisição **HEAD** feita pelo `curl -I`.

**Importante:** esse `403` não comprova, sozinho, um problema de acesso à página para requisições GET. Depois, a página de teste do Apache abriu normalmente em um navegador, então não foi preciso mudar arquivos ou permissões do servidor web.

### 7. Confirmar que a porta 80 estava escutando

```bash
sudo ss -lntp | grep ':80'
```

**Resultado resumido:**

```text
LISTEN 0 128 *:80 *:* users:(("httpd",...))
```

- `LISTEN`: o servidor espera conexões.
- `*:80`: o Apache escuta na porta 80 nas interfaces disponíveis.
- `httpd`: o processo que utiliza a porta.

### 8. Validar pelo navegador

Houve diferença nos testes feitos em dois navegadores:

- **Brave:** continuou apresentando timeout naquele momento.
- **Microsoft Edge:** abriu a página **“Test Page for the Apache HTTP Server”** pelo **IP público da EC2**.

Esse acesso confirmou que a página padrão do Apache estava disponível externamente. A diferença entre os navegadores não foi investigada até o fim, então não há uma causa confirmada para o timeout que persistiu no Brave.

## Resultado final

✅ **Objetivo principal alcançado:** o servidor Apache ficou acessível pela internet, e a página de teste carregou no Microsoft Edge.

### O que foi encontrado e corrigido

1. **Apache parado:** identificado por `systemctl status` e corrigido iniciando o serviço.
2. **Porta HTTP sem regra de entrada no Security Group:** identificada no console AWS e corrigida liberando a porta TCP 80 para o teste.
3. **Network ACL:** verificada, sem bloqueio relevante observado e sem necessidade de alterações.
4. **Porta 80 e resposta local:** confirmadas com `ss` e `curl`.
5. **Teste externo:** página de teste do Apache aberta com sucesso no Edge.

## O que aprendi

- Ter acesso SSH **não significa** que o acesso HTTP também está liberado: são portas e serviços diferentes.
- Um servidor pode estar instalado e **parado**. Primeiro precisamos verificar o serviço.
- O **Security Group** funciona como um firewall da instância e precisa permitir o acesso à porta do serviço.
- A **Network ACL** controla o tráfego da sub-rede; precisamos conferir tanto entrada quanto saída.
- `curl` e `ss` ajudam a separar problemas do serviço de possíveis problemas de rede.
- Um teste em outro navegador pode ajudar na investigação, mas é importante não adivinhar a causa sem provas.
- É melhor **validar cada etapa antes de alterar configurações**.

## Evidências recomendadas

Os resultados acima foram registrados a partir das capturas e saídas compartilhadas durante a atividade. As imagens originais não foram incluídas neste README. Caso sejam adicionadas depois, as mais úteis são:

- Apache com `inactive (dead)` e depois `active (running)`.
- Security Group antes (somente SSH) e depois (SSH + HTTP).
- Regras de entrada e saída da Network ACL.
- Comandos `curl -I` e `ss -lntp`.
- Página **Test Page for the Apache HTTP Server** aberta no navegador.

Antes de publicar capturas, ocultar dados de conta, credenciais e outras informações que não sejam necessárias.

## Cuidados finais

Este é um laboratório temporário. Em um ambiente real, revise as origens autorizadas do SSH e do HTTP, os demais controles de segurança e o ciclo de vida do servidor. Após concluir a prática, encerre o ambiente no Vocareum conforme as instruções do curso.

**Fonte:** roteiro do laboratório **266--Lab - Solucionar problemas de rede**, capturas de tela e saídas de terminal da execução de 10/11/2026.
