# 231--Lab — Editar arquivos com Vim e Nano

## Objetivo

Aprender a trabalhar com editores de texto no terminal Linux, realizando as lições iniciais do `vimtutor` e criando, salvando, modificando e descartando alterações em arquivos com **Vim** e **Nano**.

Documentação baseada no roteiro Escola da Nuvem / Vocareum e na prática registrada em **17/11/2026**.

## Ambiente

| Item | Informação |
| --- | --- |
| Plataforma | AWS / Vocareum |
| Servidor | Amazon EC2 Linux fornecida pelo laboratório |
| Acesso | SSH via Debian no WSL (Windows) |
| Usuário remoto | `ec2-user` |
| Ferramentas | `vimtutor`, `vim` e `nano` |

> A conexão SSH usou uma chave temporária de laboratório. IPs, chaves e credenciais não foram publicados.

## Passo a passo realizado

### 1. Acesso SSH e preparação

Foi estabelecida conexão SSH com a EC2 a partir do Debian no Windows. Os editores foram utilizados diretamente no terminal da atividade.

### 2. `vimtutor` — lições 1 a 3

```bash
vimtutor
```

O tutorial foi praticado até a conclusão das lições **1, 2 e 3**, incluindo os seguintes movimentos e operações:

| Comando | Para que serve |
| --- | --- |
| `h`, `j`, `k`, `l` | Mover o cursor |
| `x` | Excluir um caractere |
| `i` | Entrar no modo de inserção |
| `A` | Acrescentar texto ao fim da linha |
| `dw` | Excluir uma palavra a partir do cursor |
| `d$` | Excluir do cursor até o fim da linha |
| `dd` | Excluir a linha inteira |
| `2w`, `3e`, `d2w`, `2dd` | Usar contagens em movimentos/edições |
| `u`, `U`, `Ctrl+R` | Desfazer, restaurar alterações da linha e refazer |
| `p` | Colar o texto excluído |
| `r` | Substituir um caractere |
| `ce`, `c$` | Alterar parte do texto |
| `:q!` | Sair sem salvar alterações |

No Vim, é importante diferenciar o **modo normal**, usado para navegação e comandos, do **modo de inserção**, usado para digitar texto.

### 3. Criar e salvar o arquivo `helloworld` no Vim

```bash
vim helloworld
```

Após pressionar `i`, foi digitado:

```text
Olá, mundo!
Este é meu primeiro arquivo no Linux e estou editando-o no Vim!
```

A gravação foi feita com `Esc` e `:wq` (salvar e sair).

O arquivo foi reaberto, e o texto abaixo foi acrescentado durante um teste:

```text
Aprendi a criar um arquivo, editá-lo e salvá-lo também!
```

Em seguida, foi usado `:q!` para **descartar** a alteração e verificar, ao reabrir o arquivo, que a nova linha não havia sido salva.

O desafio adicional também abordou:

```vim
dd
u
:w
```

- `dd`: apaga uma linha.
- `u`: desfaz a última alteração.
- `:w`: grava o arquivo sem sair do editor.

### 4. Criar o arquivo `cloudworld` no Nano

```bash
nano cloudworld
```

O texto inserido foi:

```text
Desta vez estamos usando nano! Podemos simplesmente começar a digitar! Não é necessário modo de inserção.
```

Para concluir:

1. `Ctrl+O` e `Enter`: salvar o arquivo.
2. `Ctrl+X`: sair do Nano.
3. `nano cloudworld`: reabrir e verificar o texto salvo.

**Resultado observado:** o arquivo foi criado, salvo e conferido.

## Vim x Nano

| Vim | Nano |
| --- | --- |
| Possui modos de edição | Permite começar a digitar diretamente |
| Usa `i` para inserir texto | Não exige alternar para um modo de inserção |
| `:wq` salva e sai | `Ctrl+O` salva; `Ctrl+X` sai |
| `:q!` sai descartando alterações | Solicita confirmação ao sair com mudanças não salvas |

## O que aprendi

- Editar arquivos pelo terminal é útil para administrar servidores Linux remotamente.
- No Vim, comandos como `i`, `Esc`, `:wq` e `:q!` são essenciais para controlar se as mudanças são persistidas.
- No Nano, os atalhos exibidos na interface tornam a edição inicial mais simples.
- Salvar, sair e reabrir o arquivo é uma forma prática de confirmar o resultado.

## Segurança e observações

Não publicar conteúdos reais de `/var/log/secure`, credenciais, chaves SSH ou dados sensíveis. Os exemplos acima usam apenas texto didático e comandos praticados. As imagens originais não foram incluídas.

**Fonte:** roteiro **231--Lab - Editar arquivos**, Escola da Nuvem / Vocareum, e registros compartilhados da prática.
