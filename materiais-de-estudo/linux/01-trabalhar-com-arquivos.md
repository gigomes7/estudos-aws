# Guia de estudo — Trabalhar com arquivos no Linux

**Escola da Nuvem | Princípios básicos do Linux**  
Resumo autoral baseado nos slides e anotações compartilhados. Exemplos adicionais estão sinalizados como exemplos, não como resultados do laboratório.

## Objetivos da aula
Localizar arquivos, pesquisar palavras dentro de arquivos, comparar conteúdos, entender links e inodes e arquivar ou compactar dados.

## 1. Localização e pesquisa
### hash
O comando interno **hash** do Bash memoriza caminhos de executáveis já encontrados. Ele não verifica a integridade de arquivos.

~~~bash
hash
hash -r
~~~

O primeiro mostra a tabela de comandos armazenados; o segundo limpa a tabela.

### cksum
Calcula uma soma de verificação (CRC) e mostra também o tamanho do arquivo:

~~~bash
cksum documento.txt
~~~

**Cuidado:** CRC ajuda a detectar alterações acidentais, mas **não** substitui SHA-256 para verificação de autenticidade contra adulterações intencionais.

### find — procurar arquivos por propriedades

~~~bash
find . -name "relatorio.csv"
find . -iname "*.csv"
find /home -type f -name "*.txt"
find . -type d
find . -user ec2-user
find . -size +10M
find . -mtime -7
find . -maxdepth 2 -type f
~~~

| Opção | Significado |
| --- | --- |
| -name | Busca pelo nome, diferenciando maiúsculas/minúsculas |
| -iname | Busca pelo nome sem diferenciar maiúsculas/minúsculas |
| -type f | Apenas arquivos regulares |
| -type d | Apenas diretórios |
| -user | Filtra pelo proprietário |
| -size | Filtra pelo tamanho |
| -mtime | Filtra pela data de modificação |
| -maxdepth | Limita a profundidade da busca |

O find também aceita ações:

~~~bash
find . -name "*.log" -fprint encontrados.txt
find . -name "*.log" -exec ls -l {} \;
~~~

**Atenção:** a opção -delete apaga os resultados encontrados. Não executar em locais importantes sem primeiro conferir a seleção.

### grep — procurar texto dentro de arquivos

~~~bash
grep "erro" sistema.log
grep -i "erro" sistema.log
grep -n "erro" sistema.log
grep -r "erro" ./logs
grep -l "erro" *.log
grep -c "erro" sistema.log
~~~

**Memorize:** find procura arquivos; grep procura **conteúdo**.

### diff — comparar conteúdo de arquivos

~~~bash
diff versao1.txt versao2.txt
~~~

Exibe diferenças entre os arquivos de texto.

## 2. Inodes e links

**Inode** é uma estrutura de metadados do sistema de arquivos: registra informações como proprietário, permissões e localização dos dados. O nome do arquivo é armazenado na entrada de diretório, que faz referência ao inode.

- **Hard link:** outro nome para o mesmo inode. Criado com ln. Normalmente não atravessa sistemas de arquivos e não aponta para diretórios.
- **Link simbólico:** arquivo especial que aponta para um caminho. Criado com ln -s. Pode apontar para diretórios e até para destinos inexistentes; se o destino sumir, o link fica quebrado.

~~~bash
ln original.txt copia-link.txt
ln -s original.txt atalho.txt
ls -li
ls -l
~~~

**Questão comum de prova:** remover o arquivo original não destrói o conteúdo enquanto existir um hard link válido. Já um link simbólico pode ficar quebrado após a exclusão ou movimentação de seu destino.

## 3. Arquivamento e compactação
### tar — agrupar arquivos

~~~bash
tar -cvf backup.tar CompanyA/
tar -czvf backup.tar.gz CompanyA/
tar -tzf backup.tar.gz
tar -xzvf backup.tar.gz
~~~

| Opção | Função |
| --- | --- |
| -c | Criar arquivo tar |
| -x | Extrair conteúdo |
| -t | Listar conteúdo |
| -v | Mostrar o que está sendo processado |
| -f | Indicar o nome do arquivo |
| -z | Usar gzip |

**Dica importante:** quando as opções curtas forem agrupadas sem sinais adicionais, é mais seguro deixar f ao final das letras de opção antes do nome do arquivo, por exemplo tar -czvf backup.tar.gz pasta; tar -cfv pode interpretar v como argumento de f, dependendo do formato.

### gzip e zip

~~~bash
gzip arquivo.log
gunzip arquivo.log.gz
zip -r backup.zip pasta/
unzip backup.zip
~~~

**Diferença:** gzip normalmente comprime um fluxo/arquivo; tar agrega vários arquivos e pode usar gzip. Zip combina empacotamento e compressão no mesmo formato.

## 4. Exemplo prático de backup (exemplo ilustrativo)

~~~bash
pwd
tar -czvf backup.tar.gz CompanyA/
tar -tzf backup.tar.gz
mkdir -p arquivos
mv backup.tar.gz arquivos/
ls -lh arquivos/
~~~

Antes de usar comandos de extração ou remoção em servidores importantes, confira nomes, caminhos e permissões.

## 5. Revisão rápida
1. Qual a diferença entre find e grep?
2. Qual a diferença entre hard link e link simbólico?
3. Que comando lista o conteúdo de um tar.gz sem extrair?
4. Por que cksum não substitui um hash criptográfico?
5. O que significa .. em um caminho Linux?

**Respostas:** 1) Arquivos por propriedades vs. texto dentro de arquivos. 2) Mesmo inode vs. referência a caminho. 3) tar -tzf arquivo.tar.gz. 4) CRC não é resistente a adulterações intencionais. 5) Diretório pai.

## Laboratório relacionado
[Lab 235 — Trabalhar com arquivos](../../laboratorios/235-trabalhar-arquivos-linux/README.md).