# Guia de estudo — Gerenciamento de software no Linux

**Escola da Nuvem | Princípios básicos do Linux**  
Resumo autoral dos slides compartilhados sobre pacotes, repositórios, RPM, YUM, DPKG, APT, compilação, wget, curl e AWS CLI.

## 1. O que é gerenciamento de software?
É o processo de instalar, atualizar, inventariar e remover aplicativos e componentes de um sistema.

- **Pacote de software:** reúne código, metadados, arquivos e instruções necessárias à instalação; os slides destacam código pré-compilado, documentação e instruções de instalação.
- **Gerenciador de pacotes:** instala e remove software e, em ferramentas de alto nível, também trata dependências.
- **Repositório:** local remoto ou local que disponibiliza pacotes.

**Analogia:** repositório = catálogo; pacote = caixa do aplicativo; gerenciador = ferramenta que localiza, instala e mantém.

## 2. Famílias de distribuições
| Família | Arquivo | Ferramenta de baixo nível | Ferramenta de alto nível |
| --- | --- | --- | --- |
| Red Hat, CentOS, Amazon Linux 2 | .rpm | RPM | YUM |
| Debian e Ubuntu | .deb | DPKG | APT |

**Nota de atualização:** o Amazon Linux 2023 usa DNF como gerenciador padrão, mantendo yum como comando compatível. Já os laboratórios com Amazon Linux 2 utilizam YUM.

RPM e DPKG gerenciam pacotes de formatos específicos; YUM, DNF e APT facilitam consultas em repositórios e tratamento de dependências.

## 3. Comandos YUM no Amazon Linux 2
~~~bash
sudo yum install tree -y
sudo yum update
sudo yum update httpd
sudo yum remove tree
yum list installed
yum list installed | grep ssh
yum repolist
~~~

| Comando | Finalidade |
| --- | --- |
| yum install pacote | Instala um pacote |
| -y | Confirma automaticamente |
| yum update pacote | Atualiza o pacote informado |
| yum update | Atualiza pacotes disponíveis |
| yum list installed | Lista os pacotes instalados |
| grep ssh | Filtra linhas contendo ssh |
| yum remove pacote | Remove um pacote |
| yum repolist | Consulta repositórios habilitados |

O exemplo da aula usa yum list installed com grep ssh para mostrar pacotes instalados relacionados ao SSH.

Também se pode consultar diretamente um pacote específico, por exemplo com yum list installed httpd.

## 4. Comandos APT no Debian e Ubuntu
~~~bash
sudo apt update
sudo apt install psmisc -y
sudo apt upgrade
sudo apt remove psmisc
apt list --installed
~~~

- **apt update:** atualiza a lista de versões de software disponíveis; não instala essas atualizações automaticamente.
- **apt upgrade:** atualiza pacotes instalados.
- **apt install / apt remove:** instala ou remove softwares.

**Exemplo do WSL:** no Debian, o comando pstree pode ser instalado pelo pacote psmisc.

## 5. Repositórios
Tipos apresentados nos slides:
- **Online:** gerenciados por fornecedor ou mantenedor.
- **Servidor interno:** gerenciado pela organização.
- **Local:** arquivos de pacote em discos ou mídias locais.

Os slides mencionam **amzn2-core** (pacotes principais do Amazon Linux 2) e **amzn2extra-docker** (pacotes relacionados ao Docker) como exemplos.

As informações de repositórios são armazenadas em arquivos de configuração, como /etc/yum.conf e arquivos sob /etc/yum.repos.d/ nos sistemas baseados em YUM.

## 6. Instalação pelo código-fonte
Quando não há pacote pronto ou o software exige compilação, o fluxo típico é:

1. Baixar o código-fonte, frequentemente em um tarball.
2. Extrair os arquivos.
3. Compilar com ferramentas como GCC.
4. Instalar conforme as instruções do projeto.

~~~bash
tar -xzf codigo-fonte.tar.gz
~~~

Evite executar scripts desconhecidos de instalação diretamente em servidores importantes.

## 7. wget e curl
| Ferramenta | Uso |
| --- | --- |
| wget | Baixar arquivos, podendo fazer download recursivo e retomadas |
| curl | Transferir dados por URL, testar HTTP/HTTPS e consumir APIs |

~~~bash
wget https://example.com/arquivo.zip
curl -o arquivo.zip https://example.com/arquivo.zip
curl -I https://example.com
~~~

**Dica de prova:** o argumento -o no curl indica onde salvar o resultado. A afirmação dos slides de que curl só pode fazer uma transferência por vez é simplificada: a ferramenta suporta múltiplas transferências.

## 8. Exemplo da aula: instalar a AWS CLI v2 no Linux x86-64
~~~bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
~~~

O procedimento baixa o instalador oficial, extrai, instala e verifica a versão. Antes de aplicar, confira a arquitetura da máquina e se já existe uma instalação.

**Segurança:** nunca publique arquivos PEM, AWS access keys, secret keys, session tokens, arquivos de credenciais ou capturas com valores sensíveis. Quando disponíveis, utilize IAM Roles para instâncias EC2 em vez de chaves copiadas manualmente.

## 9. Comparativo de revisão
| Pergunta | Resposta |
| --- | --- |
| Quais três partes de um pacote os slides destacam? | Código pré-compilado, documentação, instruções de instalação |
| Qual formato usa Debian? | .deb |
| Qual ferramenta de alto nível usa Debian? | APT |
| Qual gerenciador foi usado nas práticas de Amazon Linux 2? | YUM |
| Qual gerenciador padrão do Amazon Linux 2023? | DNF |
| Como atualizar um pacote com YUM? | yum update nome-do-pacote |
| Como listar pacotes instalados? | yum list installed |
| Como filtrar pacotes SSH? | yum list installed e grep ssh |
| O que é um repositório? | Local de distribuição dos pacotes |

## 10. Perguntas de revisão
1. Qual é a diferença entre pacote e repositório?
2. Qual a diferença entre RPM e YUM?
3. apt update já atualiza os programas?
4. Qual é a função do parâmetro -o no curl?
5. Quais são os passos típicos da compilação pelo código-fonte?

**Respostas:** 1) Pacote é o conteúdo instalável, repositório é onde está disponível. 2) RPM gerencia pacotes de formato rpm; YUM facilita repositórios e dependências. 3) Não, atualiza o índice. 4) Define arquivo de saída. 5) Baixar, extrair, compilar e instalar.

## Laboratório relacionado
[Lab 243 — Gerenciamento de software](../../laboratorios/243-gerenciamento-software-linux/README.md).

**Fonte conceitual:** conteúdo dos slides da Escola da Nuvem/AWS re/Start enviado na conversa, resumido em palavras próprias e com pequenas correções técnicas.