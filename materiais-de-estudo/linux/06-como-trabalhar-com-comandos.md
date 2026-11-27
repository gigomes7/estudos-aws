# Guia de estudo 06 — Como trabalhar com comandos no Linux

**Escola da Nuvem / AWS re/Start | Princípios básicos do Linux**

Material autoral organizado a partir dos slides e anotações enviados na conversa. O objetivo é entender **espaços e aspas no Bash, metacaracteres, redirecionamentos, pipes, substituição de comandos e ferramentas de texto**. Os comandos apresentados são exemplos de estudo, não novas execuções em uma EC2.

## 1. Mapa rápido da aula

| Tema | Ideia-chave |
| --- | --- |
| Espaços e aspas | Em geral, espaços separam argumentos; aspas protegem um texto com espaços |
| Curingas | *, ? e [...] selecionam nomes de arquivos por padrões |
| Substituição de comandos | $(comando) coloca a saída de um comando em outro |
| Encadeamento | ; separa comandos na mesma linha |
| Redirecionamento | >, >>, <, 2> e 2>> controlam entrada/saída |
| Pipe | \| envia a saída de um comando para a entrada de outro |
| tee | Mostra a saída na tela e também salva em arquivo |
| grep | Busca linhas que combinem com um padrão |
| cut | Extrai posições ou campos |
| sed | Transforma texto por regras |
| sort / uniq | Ordenam linhas e ajudam a agrupar repetições |
| awk | Seleciona campos, filtra registros e transforma dados |

## 2. Espaços e aspas no Bash

No Bash, os espaços normalmente **separam os argumentos** de um comando.

Exemplo dos slides, apresentado para estudo (não executar sem necessidade, pois altera dados de usuário):

~~~bash
sudo usermod -c "This is a dev user" jdoe
~~~

- usermod altera informações de uma conta existente.
- -c define o comentário associado ao usuário.
- O texto entre aspas duplas é passado como **um argumento**, apesar dos espaços.

Se as aspas forem omitidas, "This", "is", "a", "dev" e "user" serão interpretados como argumentos separados e o comando não terá a sintaxe esperada.

**Exemplo seguro para testar:**

~~~bash
echo "Meu primeiro script Bash"
~~~

**Aspas simples versus duplas:**
- "texto": protege espaços, mas permite certas expansões, como $(pwd) e $HOME.
- 'texto': preserva literalmente expressões como $HOME e $(pwd).

~~~bash
echo "Diretório: $(pwd)"
echo 'Diretório: $(pwd)'
~~~

## 3. Metacaracteres e curingas

São caracteres com significado especial para o shell. Na busca por nomes de arquivos, os três padrões centrais da aula são:

| Padrão | Significado | Exemplo |
| --- | --- | --- |
| * | Zero ou mais caracteres | ls *.txt |
| ? | Exatamente um caractere | ls sales_201?.txt |
| [abc] | Exatamente um entre a, b ou c | ls log_[abc].txt |
| [a-e] | Um caractere de a até e | ls log_[a-e].txt |
| [a-zA-Z0-9] | Uma letra ou um número | ls log_[a-zA-Z0-9].txt |

### Exemplo do slide: asterisco (*)

~~~bash
ls *.txt
cp *.txt documents/
~~~

Os arquivos terminados em .txt são listados ou copiados. Outros tipos, como .png, não combinam com o padrão.

### Exemplo do slide: interrogação (?)

~~~bash
ls sales_201?.txt
~~~

O padrão combina com sales_2017.txt, sales_2018.txt, sales_2019.txt, mas não com sales_2020.txt.

**Atenção:** o slide usa rm sales_201?.txt. Esse comando **apaga** todos os arquivos correspondentes. Em exercícios, use ls sales_201?.txt primeiro para conferir a seleção.

### Exemplo do slide: colchetes

~~~bash
ls log_[abc].txt
ls log_[a-e].txt
ls log_[a-zA-Z][0-9]
ls log_?[a-zA-Z0-9]*
~~~

- [abc] seleciona um caractere: a, b ou c.
- [a-e] seleciona uma letra nesse intervalo.
- Os padrões podem ser combinados.

**Precisão importante:** o asterisco * corresponde a **zero ou mais** caracteres, não necessariamente a um ou mais.

## 4. Outros metacaracteres úteis

### Diretório pessoal (~) e diretório anterior (cd -)

~~~bash
cd ~
pwd
cd /etc
cd -
~~~

- ~ expande para o diretório pessoal do usuário.
- cd - retorna ao diretório de trabalho anterior.
- O hífen isolado não é, por si só, um atalho geral: neste caso é o argumento especial de cd.

### Separar comandos com ponto e vírgula (;)

~~~bash
date ; who ; uptime
~~~

Os três comandos são executados em sequência na mesma linha.

**Atenção:** usar ; **não depende de o comando anterior ter dado certo**. Para executar o segundo apenas se o primeiro tiver sucesso, existe && (explicação complementar, não foco do slide).

### Substituição de comandos: $(...)

~~~bash
echo "Diretório atual: $(pwd)"
echo "Data de hoje: $(date +%F)"
~~~

O Bash executa o comando interno e substitui a expressão pelo resultado. Os slides também mostram a sintaxe antiga com acentos graves (crases), mas **$(comando)** costuma ser mais legível.

**Exemplo em script:**

~~~bash
DATA=$(date +%F)
echo "Início: $DATA"
echo "Arquivo criado em $DATA"
~~~

Guardar o resultado evita executar date repetidamente; porém, se o valor precisar refletir atualizações ao longo do tempo, deve ser obtido novamente.

## 5. Entrada, saída e erros — stdin, stdout e stderr

Em um comando Linux, pense em três fluxos:

- **stdin (0):** entrada padrão; normalmente vem do teclado ou de outro fluxo.
- **stdout (1):** saída normal; normalmente aparece no terminal.
- **stderr (2):** mensagens de erro; também aparecem no terminal, mas são um fluxo separado.

| Operador | Função |
| --- | --- |
| > | Grava a saída normal em arquivo, substituindo o conteúdo anterior |
| >> | Acrescenta a saída normal ao final do arquivo |
| < | Fornece a entrada de um comando a partir de um arquivo |
| \| | Envia a saída normal do primeiro comando para a entrada do segundo |
| 2> | Grava os erros em arquivo, substituindo o conteúdo anterior |
| 2>> | Acrescenta erros ao final do arquivo |

### Exemplos apresentados na aula

~~~bash
uptime > info.txt
hostname >> info.txt
cat info.txt

# Fornecer um arquivo como entrada
less < info.txt

# Listar processos e filtrar sshd
ps -ef | grep sshd

# Salvar erros (por exemplo, "Permission denied")
find ../ -name 'p*' 2> error.log
cat error.log
~~~

**Cuidado com >:** se info.txt já existir, o shell substitui seu conteúdo. O >> preserva as linhas anteriores e acrescenta novas linhas.

**Diferença fundamental:** o pipe não salva o texto em arquivo; ele o encaminha ao próximo comando.

## 6. noclobber — proteção contra sobrescrita

Por padrão, > pode sobrescrever um arquivo existente sem perguntar. O Bash permite ativar a opção noclobber:

~~~bash
set -o noclobber
echo "Primeiro valor" > exemplo.txt
echo "Segundo valor" > exemplo.txt
~~~

No segundo redirecionamento, o Bash impede a substituição de exemplo.txt, caso ele já exista.

~~~bash
# Continuar acrescentando conteúdo
echo "Outra linha" >> exemplo.txt

# Desativar noclobber quando não precisar mais
set +o noclobber
~~~

**Nota de precisão:** noclobber é uma **opção do Bash**, não uma variável de ambiente. Trocar > por >> evita sobrescrever o arquivo, mas **não produz o mesmo resultado**: >> acrescenta conteúdo, enquanto a proteção do noclobber impede a substituição.

## 7. Pipe e processamento em etapas

O símbolo | conecta comandos.

~~~bash
ps -ef | grep sshd
ls -l /etc | less
~~~

No primeiro exemplo:
1. ps -ef lista processos.
2. | transfere a saída do ps para o próximo comando.
3. grep sshd mostra as linhas que contêm sshd.

No segundo, less recebe a listagem de /etc para visualização paginada. Pressione q para sair.

**Pipeline com três comandos, como no slide:**

~~~bash
ps -au | grep ec2-user | awk '{print $1, $2}'
~~~

Fluxo: listar processos → filtrar ec2-user → mostrar o usuário e o PID (primeiro e segundo campos, nesse formato de saída).

**Atenção:** a forma exata das colunas do ps depende das opções utilizadas.

## 8. grep — procurar texto

~~~bash
grep 'ERROR' arquivo.log
grep -i 'error' arquivo.log
ps -ef | grep sshd
sudo grep 'Failed password' /var/log/secure
~~~

- grep encontra linhas que contêm o padrão.
- -i ignora diferenças entre maiúsculas e minúsculas.
- Com |, busca na saída do comando anterior.
- Alguns logs exigem permissões elevadas para leitura.

**Exemplo em cadeia:** ps -ef | grep sshd ajuda a identificar processos associados ao serviço SSH.

## 9. tee — mostrar e salvar ao mesmo tempo

~~~bash
hostname | tee hostname.txt
~~~

O tee recebe a saída de hostname, **mostra no terminal** e **grava no arquivo**.

~~~bash
date | tee -a atividades.txt
~~~

Com -a, acrescenta ao arquivo existente. Sem -a, substitui seu conteúdo.

**Comparação de prova:**
- comando > arquivo.txt: salva a saída no arquivo.
- comando | tee arquivo.txt: salva **e** mostra na tela.

## 10. cut — extrair partes de linhas

O cut trabalha com **campos, caracteres ou bytes**, dependendo da opção.

| Opção | Para que serve |
| --- | --- |
| -d | Define o delimitador de campos |
| -f | Seleciona o(s) campo(s) |
| -c | Seleciona posições de caracteres |
| -b | Seleciona posições de bytes |

**Suponha um arquivo names.csv com uma linha como esta:**

~~~text
Ana Silva,28,Sao Paulo
~~~

~~~bash
cut -d ',' -f 1 names.csv
cut -c 1-2 names.csv
cut -b 1-5 names.csv
cut -c 1,6,7 names.csv
cut -c 4- names.csv
cut -c -3 names.csv
~~~

- O primeiro exemplo retorna o nome, porque a vírgula separa os campos.
- -c 1-2 extrai as duas primeiras posições.
- -b 1-5 extrai os primeiros cinco **bytes**.
- -c 4- extrai a partir da quarta posição.
- -c -3 extrai até a terceira posição.

**Nota:** em textos com acentos e codificações multibyte, a correspondência entre bytes e caracteres exige atenção. Os slides também incluem um erro de digitação "names.cvs"; a extensão usada nos exemplos é .csv.

## 11. sed — editar ou substituir texto na saída

~~~bash
echo "example.com page" | sed 's/page/website/'
sed 's/page/website/' example.txt
sed 's/page/website/g' example.txt
~~~

- s/antigo/novo/ faz a substituição da **primeira ocorrência por linha**.
- Com g, substitui **todas as ocorrências da linha**.
- Também existe a seleção de uma ocorrência específica, por exemplo s/page/website/5 para a quinta ocorrência.

**Importante:** sed sem -i normalmente **mostra o resultado transformado, mas não altera o arquivo original**. Isso corrige uma possível interpretação do texto da aula.

## 12. sort — ordenar linhas

~~~bash
sort names.csv
sort -r names.csv
sort -n numeros.txt
sort -u names.csv
sort -M meses.txt
sort -c names.csv
sort names.csv > names_ordenados.csv
~~~

| Opção | Significado |
| --- | --- |
| -r | Ordem inversa |
| -n | Comparação numérica |
| -u | Mantém uma ocorrência de cada linha repetida ao ordenar |
| -M | Ordenação por nomes de meses reconhecidos |
| -c | Verifica se as linhas já estão ordenadas |
| -o arquivo | Grava o resultado no arquivo escolhido |

A ordem alfabética exata pode mudar de acordo com a configuração de localidade (locale) do sistema.

## 13. uniq — tratar linhas repetidas

Os materiais complementares apresentam:

~~~bash
uniq -c logfile.log
~~~

O comando agrupa **linhas repetidas consecutivas** e mostra sua contagem. Para contar todas as linhas iguais espalhadas pelo arquivo, uma solução comum é **ordenar primeiro**:

~~~bash
sort logfile.log | uniq -c
~~~

**Pegadinha de prova:** uniq sozinho não procura repetições distantes entre si; ele compara linhas adjacentes.

## 14. awk — selecionar campos, filtrar e gerar relatórios

O awk lê dados linha por linha, permite dividir campos, aplicar condições e imprimir resultados.

**Sintaxe geral:**

~~~bash
awk -F ',' '{print $3}' names.csv
~~~

- -F ',' define vírgula como separador.
- $1 é o primeiro campo, $2 o segundo, $3 o terceiro.
- print escreve a informação selecionada.

**Exemplo fictício, simplificado:**

~~~text
Ana,28,Suporte
Bruno,42,Cloud
Clara,31,DevOps
~~~

~~~bash
# Mostrar a área (terceiro campo)
awk -F ',' '{print $3}' names.csv

# Mostrar somente nomes de pessoas com idade maior que 35
awk -F ',' '$2 > 35 {print $1}' names.csv

# Mostrar nomes nas linhas que contêm dois dígitos seguidos
awk -F ',' '/[0-9][0-9]/ {print $1}' names.csv

# Separar um endereço com @ e imprimir a parte anterior
awk -F '@' '{print $1}' emails.txt
~~~

O exemplo de idade maior que 35 retornaria Bruno na amostra acima.

**BEGIN e END:** são blocos executados antes e depois do processamento.

~~~bash
awk -F ',' 'BEGIN {print "Inicio"} {print $1} END {print "Fim"}' names.csv
~~~

**Outras opções da aula:** -f arquivo.awk para carregar um programa awk de um arquivo; -v nome=valor para definir uma variável.

## 15. Exercício prático seguro — faça no Debian/WSL

Este treino é **opcional** e não faz parte de uma execução comprovada em laboratório. Ele cria arquivos somente em uma pasta de estudo.

~~~bash
mkdir -p ~/treino-comandos
cd ~/treino-comandos
printf 'Ana,28,Suporte\nBruno,42,Cloud\nClara,31,DevOps\nAna,28,Suporte\n' > pessoas.csv
~~~

Agora experimente:

~~~bash
# Conferir arquivo
cat pessoas.csv

# Extrair os nomes
cut -d ',' -f 1 pessoas.csv

# Filtrar quem tem mais de 35 anos
awk -F ',' '$2 > 35 {print $1}' pessoas.csv

# Contar nomes repetidos
cut -d ',' -f 1 pessoas.csv | sort | uniq -c

# Substituir o termo Cloud apenas na saída
sed 's/Cloud/AWS Cloud/' pessoas.csv

# Mostrar e salvar um relatório simples
cut -d ',' -f 1 pessoas.csv | sort -u | tee nomes.txt

# Conferir arquivo salvo
cat nomes.txt
~~~

**Resultado esperado da contagem:** Ana 2 vezes, Bruno 1 vez, Clara 1 vez.

**Segurança:** o primeiro comando com > substitui pessoas.csv se ele já existir. Use uma pasta de exercícios para não modificar arquivos importantes. Evite copiar exemplos destrutivos com rm sem conferir antes com ls.

## 16. Questões de revisão (estilo prova)

1. Como passar um texto com espaços como um único argumento? 
2. Qual curinga corresponde a exatamente um caractere? 
3. Qual a diferença entre * e [abc]?
4. Qual operador acrescenta conteúdo a um arquivo sem descartar o que já estava lá?
5. Qual operador redireciona mensagens de erro para um arquivo?
6. O que faz o pipe |?
7. O que diferencia tee de >?
8. Qual comando permite obter o primeiro campo separado por vírgula?
9. sed 's/a/b/g' altera o arquivo original automaticamente?
10. Qual é a forma correta de contar repetições não consecutivas usando uniq?
11. Como mostrar o segundo campo com awk quando o separador é vírgula?
12. O que faz $(pwd) em um comando echo?

**Gabarito:**
1. Usar aspas ("texto com espaços").
2. ?.
3. * = zero ou mais caracteres; [abc] = exatamente um entre a/b/c.
4. >>.
5. 2>.
6. Envia a saída normal de um comando para a entrada do próximo.
7. tee mostra e grava; > apenas redireciona a saída para o arquivo.
8. cut -d ',' -f 1 arquivo.csv.
9. Não, sem -i ele normalmente mostra a transformação na saída.
10. sort arquivo | uniq -c.
11. awk -F ',' '{print $2}' arquivo.csv.
12. Substitui $(pwd) pelo diretório atual retornado por pwd.

## 17. Cola rápida para revisão

| Quero fazer... | Uso... |
| --- | --- |
| Proteger texto com espaços | "duas palavras" |
| Selecionar todos os .txt | *.txt |
| Selecionar um único caractere | ? |
| Selecionar uma opção de uma lista | [abc] |
| Voltar à pasta anterior | cd - |
| Reutilizar resultado de comando | $(comando) |
| Executar comandos em sequência | comando1 ; comando2 |
| Substituir o conteúdo de um arquivo | > |
| Acrescentar linhas ao arquivo | >> |
| Capturar erros | 2> |
| Enviar saída para outro comando | \| |
| Exibir e salvar saída | tee |
| Encontrar texto | grep |
| Extrair colunas/campos | cut |
| Substituir texto | sed |
| Ordenar | sort |
| Contar repetições após ordenar | sort \| uniq -c |
| Selecionar campos e aplicar condições | awk |

### Observações de precisão dos materiais

Os conceitos acima seguem a estrutura dos slides. Pequenas correções técnicas foram sinalizadas ao longo do guia:
- * corresponde a **zero ou mais** caracteres.
- uniq -c agrupa repetições **consecutivas**; geralmente usa-se sort antes.
- sed sem -i não modifica o arquivo-fonte.
- noclobber é opção do Bash; >> não é equivalente a habilitar essa proteção.
- O pipe | encaminha a saída normal de um comando, e o operador > pode sobrescrever arquivos existentes.
- Comandos como rm, usermod, atualização de pacotes e alteração de serviços exigem cuidado especial em sistemas reais.

**Fonte:** aula "Como trabalhar com comandos — Princípios básicos do Linux", slides e anotações da Escola da Nuvem/AWS re/Start enviados na conversa. Não reproduz os slides integrais nem presume que exemplos foram executados pela usuária.
