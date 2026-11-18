# 229--Lab — Criar usuários e grupos no Linux

## Objetivo

Praticar a administração básica de usuários e grupos em uma instância Amazon EC2 Linux: criar contas, organizar permissões de acesso por grupo, alternar de usuário e verificar o comportamento do `sudo`.

Este registro reúne o roteiro da Escola da Nuvem / Vocareum e as etapas efetivamente realizadas em **novembro de 2026**. Não representa uma nova execução na AWS no momento da publicação.

## Ambiente

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Servidor | Instância Amazon EC2 com Linux |
| Acesso | SSH pelo Debian no WSL (Windows) |
| Usuário administrativo do laboratório | `ec2-user` |
| Ferramentas | `useradd`, `passwd`, `groupadd`, `usermod`, `su`, `sudo`, `cat`, `cut`, `touch` |

> A chave SSH, o IP público temporário e as senhas de teste não foram incluídos neste repositório.

## Passo a passo e resultados

### 1. Acesso remoto

A conexão por SSH foi realizada a partir do Debian/WSL, usando uma chave temporária do Vocareum com permissão restrita.

```bash
chmod 400 ~/chave-do-laboratorio.pem
ssh -i ~/chave-do-laboratorio.pem ec2-user@IP_PUBLICO_TEMPORARIO
```

**Resultado observado:** acesso ao terminal da instância com o usuário `ec2-user`.

### 2. Criação e conferência dos usuários

Os comandos principais usados foram:

```bash
sudo useradd arosalez
sudo passwd arosalez
sudo cat /etc/passwd | cut -d: -f1
```

O procedimento foi repetido para os usuários definidos na atividade. Os **10 identificadores pretendidos** eram:

```text
arosalez
eowusu
jdoe
ljuan
mmajor
mjackson
nwolf
psantos
smartinez
ssarkar
```

**Observação importante da execução:** a tradução automática do roteiro apresentou os nomes `maior` e `inteligente` no lugar de `mmajor` e `smartinez`. Esses dois usuários extras também foram criados, totalizando **12 contas de exercício**. Os nomes corretos foram posteriormente considerados no procedimento.

**Aprendizado:** nomes de usuário devem ser tratados como identificadores exatos; não devem ser traduzidos.

### 3. Criação de grupos

Foram criados e conferidos os grupos:

```text
Sales
HR
Finance
Personnel
CEO
Shipping
Managers
```

Exemplos dos comandos utilizados:

```bash
sudo groupadd Sales
sudo usermod -a -G Sales arosalez
cat /etc/group
```

A associação dos usuários foi feita conforme as funções indicadas na atividade, e o usuário `ec2-user` foi adicionado aos grupos do exercício.

- `groupadd`: cria o grupo.
- `usermod -a -G`: adiciona o usuário a grupos complementares **sem substituir** as associações existentes.
- `/etc/group`: permite verificar os membros dos grupos.

### 4. Teste com outro usuário e permissões

Foi realizado login no usuário criado para o exercício:

```bash
su arosalez
pwd
touch myFile.txt
```

**Resultado observado:** o comando `touch` retornou **Permission denied**, pois `arosalez` não possuía permissão para gravar em `/home/ec2-user`.

Em seguida foi testado:

```bash
sudo touch myFile.txt
```

**Resultado observado:** o usuário `arosalez` não estava autorizado no arquivo de configuração `sudoers`, por isso a ação administrativa foi negada.

O retorno ao usuário anterior e a consulta ao registro de segurança foram feitos com:

```bash
exit
sudo cat /var/log/secure
```

O log registrou a tentativa de uso de `sudo` sem autorização.

## Resultado final

- Contas de laboratório criadas e conferidas no `/etc/passwd`.
- Grupos criados e associações verificadas no `/etc/group`.
- Teste de troca de usuário realizado com `su`.
- Bloqueio por permissão de escrita observado.
- Bloqueio do `sudo` para usuário não autorizado observado e identificado no log.

## O que aprendi

- **Usuário** é uma identidade do Linux; **grupo** reúne usuários para facilitar o controle de acesso.
- Nem todo usuário pode modificar arquivos pertencentes a outro usuário.
- Pertencer a grupos comuns não garante acesso administrativo via `sudo`.
- `/etc/passwd` registra contas, `/etc/group` registra grupos e `/var/log/secure` ajuda na auditoria de autenticação e permissões neste ambiente.
- Um roteiro traduzido automaticamente pode alterar identificadores técnicos e gerar usuários duplicados.

## Segurança

Nunca publicar senhas (mesmo as do roteiro), chaves privadas `.pem`, credenciais AWS ou logs completos com informações desnecessárias. O acesso `sudo` deve seguir o princípio do menor privilégio.

**Fonte:** roteiro **Lab 229 — usuários e grupos**, Escola da Nuvem / Vocareum, e registros da prática compartilhados durante a aula. Capturas de tela originais não foram incluídas.
