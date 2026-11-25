# Guia de estudo — Permissões de arquivos no Linux

**Escola da Nuvem | Princípios básicos do Linux**  
Material autoral consolidado dos slides enviados. Comandos apresentados para treinamento.

## Objetivo
Entender quem pode ler, modificar ou executar arquivos e diretórios, interpretar ls -l e usar chmod e chown.

## 1. Tipos de permissão
- **r = read (leitura):** ler conteúdo de um arquivo; em diretórios, listar nomes de entradas, quando combinado com as permissões necessárias.
- **w = write (escrita):** alterar conteúdo do arquivo; em diretórios, criar, mover ou remover entradas, normalmente junto com x.
- **x = execute (execução):** executar um arquivo; em diretórios, atravessar/acessar entradas pelo caminho.

**Importante:** a semântica de r, w, x é um pouco diferente para diretórios. Para acessar um arquivo dentro de uma pasta, é necessário poder atravessar seus diretórios.

## 2. Como ler o ls -l

~~~text
-rwxr-xr-x
│├─┤├─┤├─┤
│ │   │   └── Outros
│ │   └────── Grupo
│ └────────── Proprietário
└──────────── Tipo de arquivo (- = regular; d = diretório; l = link simbólico)
~~~

Três grupos de permissões, sempre nesta ordem:

| Grupo | Símbolo | Quem representa |
| --- | --- | --- |
| Proprietário | u | Dono do arquivo |
| Grupo | g | Membros do grupo associado |
| Outros | o | Demais usuários |

Exemplo:

~~~bash
ls -l documento.txt
~~~

~~~text
-rw-r--r-- 1 ec2-user ec2-user 120 Nov 25 10:00 documento.txt
~~~

O proprietário pode ler e escrever; grupo e outros podem ler; ninguém tem permissão de execução.

## 3. chmod simbólico

~~~bash
chmod u+x script.sh
chmod g+w documento.txt
chmod g-rw documento.txt
chmod o-r documento.txt
chmod u=rw,g=r,o= documento.txt
~~~

| Elemento | Significado |
| --- | --- |
| u, g, o | Proprietário, grupo, outros |
| + | Acrescenta permissão |
| - | Remove permissão |
| = | Define exatamente as permissões informadas |
| r, w, x | Leitura, escrita, execução |

**Correção didática:** chmod g=x define o grupo com permissão **somente de execução**. Não significa apenas “remover a execução”. Um usuário comum pode alterar permissões de arquivos que possui; não é exclusivo do root.

## 4. chmod numérico (octal)

| Permissão | Valor |
| --- | --- |
| r | 4 |
| w | 2 |
| x | 1 |
| rwx | 7 |
| rw- | 6 |
| r-x | 5 |
| r-- | 4 |
| --- | 0 |

O número tem três dígitos: **proprietário, grupo, outros**.

| Modo | Significado |
| --- | --- |
| 755 | Proprietário rwx; grupo r-x; outros r-x |
| 644 | Proprietário rw-; grupo r--; outros r-- |
| 700 | Proprietário rwx; ninguém mais acessa |
| 400 | Somente leitura para o proprietário |
| 764 | Proprietário rwx; grupo rw-; outros r-- |
| 777 | Todos podem ler, escrever e executar: geralmente inseguro |

~~~bash
chmod 400 arquivo.txt
chmod 644 documento.txt
chmod 755 script.sh
~~~

**Princípio do privilégio mínimo:** conceda somente os acessos necessários, pelo tempo e para as pessoas que precisarem. Evite chmod 777 sem uma justificativa muito específica.

## 5. chown — proprietário e grupo

~~~bash
sudo chown maria documento.txt
sudo chown maria:Finance documento.txt
sudo chown :Finance documento.txt
sudo chown -R maria:Finance pasta/
~~~

O comando chown modifica o proprietário e/ou o grupo associado. Em geral, a troca de proprietário exige permissões administrativas. A opção -R aplica a operação recursivamente à árvore, e deve ser usada com cuidado.

**Diferença para prova:**
- chmod muda **permissões**;
- chown muda **proprietário/grupo**;
- ls -l **consulta** os dois.

## 6. Permissões iniciais e sudo

Ao criar um arquivo ou pasta, as permissões finais normalmente dependem das permissões base e do **umask**. Exemplos comuns: arquivos 644 e diretórios 755, mas os valores variam conforme o umask e o contexto.

O usuário root possui privilégios administrativos. O comando sudo executa uma operação com permissões elevadas, quando autorizado; ele não transforma automaticamente todas as sessões do usuário em root.

**Nota:** não use sudo indiscriminadamente em diretórios compartilhados; isso pode gerar arquivos com dono root e causar erros posteriores para usuários comuns.

## 7. Exercício demonstrado nos slides

~~~bash
touch file_1
ls -l file_1
chmod 400 file_1
ls -l file_1
~~~

Resultado esperado após chmod 400: permissões de arquivo regular -r-------- (somente leitura pelo proprietário).

## 8. Revisão
1. Quem pode alterar um arquivo com modo -rw-r--r--?
2. Qual modo octal representa rwxr-xr-x?
3. O que faz chmod g+w?
4. Qual comando altera o dono do arquivo?
5. Por que chmod 777 não é recomendado?

**Gabarito:** 1) Proprietário, nas permissões usuais. 2) 755. 3) Adiciona escrita para o grupo. 4) chown. 5) Dá leitura, escrita e execução a todos.

## Laboratório relacionado
[Lab 237 — Gerenciar permissões](../../laboratorios/237-gerenciar-permissoes-arquivos/README.md).