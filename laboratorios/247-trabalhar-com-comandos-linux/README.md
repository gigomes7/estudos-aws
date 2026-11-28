# 247 — Trabalhar com comandos no Linux

**Data de realização:** 27/11/2026  
**Ambiente:** instância EC2 temporária do Vocareum, região us-west-2 (Oregon), acessada por SSH a partir do Debian WSL.  
**Status:** tarefas práticas realizadas e encerramento do laboratório confirmado pela usuária.

## Objetivo

Praticar o uso de comandos Linux para exibir e salvar informações, ordenar linhas, encadear comandos, filtrar resultados, extrair campos e substituir caracteres em arquivos de texto.

## Tarefa 1 — Conexão à instância EC2

A chave privada PEM do laboratório foi localizada no Windows, copiada para a pasta pessoal no Debian WSL e teve suas permissões restringidas:

~~~bash
chmod 400 ~/CHAVE_DO_LABORATORIO.pem
~~~

A conexão SSH à instância temporária foi realizada com sucesso:

~~~bash
ssh -i ~/CHAVE_DO_LABORATORIO.pem ec2-user@<IP_PUBLICO_DA_INSTANCIA>
~~~

**Confirmação:** a sessão exibiu o prompt do usuário ec2-user. O diretório de trabalho foi validado com pwd como /home/ec2-user.

> O nome da chave e o IP público estão representados por valores ilustrativos. Nenhuma chave privada ou credencial foi publicada.

## Tarefa 2 — Usar o comando tee

### Mostrar o hostname e salvar em arquivo

~~~bash
hostname | tee file1.txt
~~~

**Resultado observado (nome da instância anonimizado):**

~~~text
ip-10-0-10-XX.us-west-2.compute.internal
~~~

O comando hostname gera o nome da máquina; o pipe (|) transfere esse resultado para tee; tee mostra a informação no terminal e a grava em file1.txt.

### Confirmar a criação do arquivo

~~~bash
ls
~~~

**Saída confirmada:**

~~~text
companyA  file1.txt
~~~

**Aprendizado:** tee arquivo.txt sobrescreve o arquivo de destino; tee -a arquivo.txt acrescenta texto sem descartar as linhas anteriores.

## Tarefa 3 — Usar sort e pipe

### Criar o arquivo test.csv

~~~bash
cat > test.csv
~~~

As cinco linhas foram digitadas conforme o roteiro:

~~~text
Fábrica, 1, Paris
Loja, 2, Dubai
Fábrica, 3, Brasília
Loja, 4, Argel
Fábrica, 5, Tóquio
~~~

Em seguida, Ctrl+D encerrou a entrada. O comando ls confirmou a existência de test.csv:

~~~text
companyA  file1.txt  test.csv
~~~

### Ordenar o arquivo

~~~bash
sort test.csv
~~~

**Saída real:**

~~~text
Fábrica, 1, Paris
Fábrica, 3, Brasília
Fábrica, 5, Tóquio
Loja, 2, Dubai
Loja, 4, Argel
~~~

**Aprendizado:** sort organiza a saída em ordem de classificação padrão. Neste arquivo, as linhas começando por Fábrica foram exibidas antes das linhas começando por Loja. Sem redirecionamento ou opções de gravação, o arquivo original não é modificado.

### Filtrar a fábrica em Paris

~~~bash
cat test.csv | grep Paris
~~~

**Saída real:**

~~~text
Fábrica, 1, Paris
~~~

**Aprendizado:**
- cat test.csv lê o conteúdo do arquivo;
- | envia a saída ao próximo comando;
- grep Paris seleciona somente as linhas que contêm Paris.

> **Correção pontual do roteiro:** o comando transcrito como find | grep Paris test.csv não demonstra o pipeline descrito. Para filtrar o conteúdo do arquivo por meio do pipe, foi executado cat test.csv | grep Paris, com resultado confirmado. Para a mesma consulta, também seria válido grep Paris test.csv.

## Tarefa 4 — Usar cut

### Criar cities.csv

O arquivo foi criado com:

~~~bash
cat > cities.csv
~~~

Conteúdo do exercício:

~~~text
Dallas, Texas
Seattle, Washington
Los Angeles, Califórnia
Atlanta, Geórgia
Nova Iorque, Nova Iorque
~~~

Um primeiro teste com cat cities.csv retornou No such file or directory, pois o arquivo ainda não havia sido criado. Após criar o arquivo com o redirecionamento > e encerrar a entrada com Ctrl+D, a leitura de cities.csv foi confirmada.

### Extrair apenas os nomes das cidades

~~~bash
cut -d ',' -f 1 cities.csv
~~~

**Saída real:**

~~~text
Dallas
Seattle
Los Angeles
Atlanta
Nova Iorque
~~~

**Aprendizado:**
- -d ',' define a vírgula como delimitador;
- -f 1 seleciona o primeiro campo de cada linha;
- cut somente exibe o resultado e não regrava o arquivo original.

## Desafio adicional — Substituição de texto com sed

### Substituir a primeira vírgula em cities.csv

~~~bash
sed 's/,/./' cities.csv
~~~

**Saída observada:**

~~~text
Dallas. Texas
Seattle. Washington
Los Angeles. Califórnia
Atlanta. Geórgia
Nova Iorque. Nova Iorque
~~~

A usuária observou o prompt junto à última linha, porque o arquivo não apresentava quebra de linha final. Isso não impediu a substituição de texto.

### Substituir a primeira vírgula em test.csv

~~~bash
sed 's/,/./' test.csv
~~~

**Saída real:**

~~~text
Fábrica. 1, Paris
Loja. 2, Dubai
Fábrica. 3, Brasília
Loja. 4, Argel
Fábrica. 5, Tóquio
~~~

**Aprendizado:** a expressão s/,/./ substitui **somente a primeira vírgula de cada linha** por um ponto. Uma segunda vírgula permanece. Como o sed foi executado sem -i, os arquivos originais não foram alterados.

## Resumo dos comandos praticados

| Comando | Uso |
| --- | --- |
| pwd | Confere a pasta atual |
| chmod 400 chave.pem | Restringe leitura da chave ao proprietário |
| ssh -i ... | Acessa a EC2 por SSH usando uma chave privada |
| hostname | Mostra o nome do computador |
| hostname \| tee file1.txt | Mostra e salva o hostname |
| ls | Lista arquivos e pastas |
| cat > test.csv | Recebe texto e o grava em arquivo; Ctrl+D finaliza |
| sort test.csv | Ordena linhas e mostra o resultado |
| cat test.csv \| grep Paris | Filtra as linhas que contêm Paris |
| cat cities.csv | Mostra conteúdo de arquivo existente |
| cut -d ',' -f 1 cities.csv | Mostra o primeiro campo separado por vírgula |
| sed 's/,/./' cities.csv | Substitui a primeira vírgula de cada linha |
| sed 's/,/./' test.csv | Aplica a mesma transformação no outro arquivo |

## Resultado e validação

**Confirmado durante a prática:**
1. Acesso à EC2 e verificação de /home/ec2-user.
2. O tee mostrou o hostname na tela e criou file1.txt.
3. O arquivo test.csv foi criado, localizado e ordenado.
4. O pipe com grep encontrou a entrada Fábrica, 1, Paris.
5. O arquivo cities.csv foi criado e consultado.
6. O cut extraiu os cinco nomes de cidades.
7. O sed trocou a primeira vírgula por ponto nos dois arquivos.
8. A usuária confirmou o encerramento do laboratório no Vocareum.

## Perguntas rápidas de revisão

1. O que diferencia tee de um redirecionamento simples usando >?
2. Qual opção do tee acrescenta conteúdo a um arquivo?
3. O que significa | em um pipeline?
4. O sort modifica test.csv ao ser executado sem redirecionamento?
5. Em cut -d ',' -f 1, qual é a função de -d e -f?
6. O comando sed 's/,/./' troca todas as vírgulas da linha?

**Respostas:** (1) tee mostra na tela e salva em arquivo; (2) -a; (3) conecta a saída do primeiro comando à entrada do próximo; (4) não; (5) -d define o delimitador e -f escolhe o campo; (6) não, somente a primeira de cada linha.

## Material de estudo relacionado

[Guia 06 — Como trabalhar com comandos no Linux](../../materiais-de-estudo/linux/06-como-trabalhar-com-comandos.md)

**Fonte:** roteiro do Lab 247 — Trabalhar com comandos fornecido pela usuária e resultados de terminal compartilhados no chat. Este documento descreve a prática realizada, sem reexecutar comandos ou instâncias na AWS durante a publicação.
