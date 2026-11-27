# Guia de estudo — Gerenciamento de arquivos de log no Linux

**Escola da Nuvem | AWS re/Start | Linux**
Resumo autoral dos slides e anotações compartilhados em 26/11/2026. Relacionado ao Lab 245.

## 1. O que são logs?

**Logs** são registros de acontecimentos em um sistema operacional, aplicativo ou serviço. Eles ajudam a entender **o que aconteceu, quando, onde e, quando registrado, qual usuário ou serviço estava envolvido**.

**Tipos destacados na aula:**
- Logs do sistema: inicialização, desligamento e outros eventos gerais.
- Logs de eventos: por exemplo, login e logout.
- Logs de aplicativos: inicialização, ações e erros.
- Logs de serviços: atividades de serviços, como Apache/HTTPD e SSH.

**Exemplo prático:** um usuário não consegue entrar por SSH. O administrador consulta logs de autenticação para identificar falhas e origem da tentativa.

## 2. Por que são importantes?

Os slides destacam quatro aplicações:
1. **Auditoria de segurança:** investigar acessos e eventos suspeitos.
2. **SLAs (acordos de nível de serviço):** registrar quando ocorreu um problema e apoiar a resposta nos prazos combinados.
3. **Resolução proativa de problemas:** encontrar sinais de falha antes que prejudiquem os usuários.
4. **Exigências do setor:** atender requisitos operacionais e de conformidade.

**Atenção:** logs podem incluir dados pessoais, IPs e informações sensíveis. Controle as permissões e evite compartilhar registros sem revisão.

## 3. Onde os logs ficam?

**Diretório tradicional: /var/log**

| Local / arquivo | Utilidade |
| --- | --- |
| /var/log/ | Diretório que costuma armazenar logs do sistema e de serviços |
| /var/log/secure | Autenticação em sistemas da família Red Hat, como CentOS e Amazon Linux 2 |
| /var/log/auth.log | Autenticação em sistemas como Debian/Ubuntu |
| /var/log/messages | Mensagens gerais em diversas distribuições da família Red Hat |
| /var/log/syslog | Mensagens gerais em diversas distribuições Debian/Ubuntu |
| /var/log/boot.log | Eventos de inicialização, quando presente |
| /var/log/kern.log | Mensagens do kernel, quando presente |
| /var/log/maillog | Logs de e-mail em algumas distribuições |
| /var/log/httpd/ | Arquivos do servidor Apache em algumas distribuições Red Hat |
| /var/log/apache2/access.log | Requisições HTTP ao Apache em diversas distribuições Debian |
| /var/log/yum.log | Histórico de alterações em pacotes pelo YUM, quando disponível |
| /var/log/lastlog | Registro binário consultado pelo comando lastlog |

Os caminhos **variam por distribuição e configuração**. Algumas máquinas armazenam parte dos eventos no journal do systemd, acessível por journalctl.

**Exemplos exibidos nos slides:** /var/log/yum.log (pacotes instalados/atualizados) e arquivos sob /var/log/httpd/ (servidor web).

## 4. Níveis de gravidade do Syslog

**Regra principal: quanto menor o número, maior a gravidade.**

| Nível | Nome | Significado simplificado |
| --- | --- | --- |
| 0 | EMERGENCY | Sistema inutilizável ou emergência |
| 1 | ALERT | Precisa de ação imediata |
| 2 | CRITICAL | Condição crítica |
| 3 | ERROR | Erro |
| 4 | WARNING | Aviso |
| 5 | NOTICE | Evento normal, mas relevante |
| 6 | INFORMATIONAL | Informações de rotina |
| 7 | DEBUG | Informações de depuração |

**Correção importante de um trecho dos slides:** ao usar WARNING (nível 4) como limite máximo de severidade numérica, em regra são incluídos os níveis **0 a 4**, e não NOTICE/INFO/DEBUG (5 a 7). A configuração real depende da ferramenta e do serviço. Não existe um nível de log padrão universal para todos os programas.

## 5. Comandos para ler logs

| Comando | O que faz |
| --- | --- |
| cat arquivo.log | Mostra o conteúdo inteiro de uma vez |
| less arquivo.log | Permite navegar pelo conteúdo; pressione q para sair |
| more arquivo.log | Exibe por páginas |
| head arquivo.log | Mostra as primeiras 10 linhas |
| tail arquivo.log | Mostra as últimas 10 linhas |
| head -n 5 arquivo.log | Primeiras 5 linhas |
| tail -n 5 arquivo.log | Últimas 5 linhas |
| tail -f arquivo.log | Acompanha novas linhas conforme são adicionadas |
| grep 'ERROR' arquivo.log | Filtra linhas que contêm ERROR |

~~~bash
# Primeiro e último trecho de um log (exemplo do slide)
sudo head -n 5 /var/log/yum.log
sudo tail -n 5 /var/log/yum.log

# Filtrar entradas que mencionem o servidor HTTP
sudo grep 'httpd' /var/log/yum.log

# Procurar a palavra "error" independentemente de maiúsculas/minúsculas
grep -i 'error' arquivo.log

# Acompanhar novos registros e filtrar a palavra "error"
tail -f arquivo.log | grep -i --line-buffered 'error'
~~~

**Pipe |:** envia a saída de um comando como entrada de outro, por exemplo:

~~~bash
sudo tail /var/log/yum.log | grep 'httpd'
~~~

**Observação:** tail -f continua acompanhando novas linhas; não se limita às últimas dez linhas. Ctrl+C interrompe o acompanhamento. O grep sem -i diferencia letras maiúsculas e minúsculas.

## 6. Filtrar e registrar ocorrências com grep

Exemplos derivados da aula:

~~~bash
# Procurar tentativas malsucedidas em um arquivo de exemplo do laboratório
sudo grep 'Failed password' /tmp/log/secure

# Exportar apenas linhas que contenham a palavra LOGIN
sudo grep 'LOGIN' /tmp/log/secure > SharedFolders/logins.csv
~~~

O sinal **>** cria ou substitui o conteúdo do arquivo de destino (cuidado para não apagar dados existentes). Um arquivo com extensão .csv só será um CSV bem estruturado se as linhas forem formatadas adequadamente; exportar texto bruto de logs não garante esse formato.

## 7. lastlog — Último login de cada usuário

~~~bash
sudo lastlog
lastlog -u ec2-user
lastlog -t 1
~~~

- **sudo lastlog:** mostra o último login registrado dos usuários.
- **lastlog -u ec2-user:** restringe a pesquisa ao usuário ec2-user.
- **lastlog -t 1:** exibe entradas cujo último login foi registrado no último dia.
- **Never logged in:** não há um login anterior registrado para aquele usuário nessa base.

**Diferença que apareceu no quiz:**
- lastlog = último login de cada usuário.
- last = histórico de sessões registradas (não apenas o último login).
- users = usuários conectados no momento.

As informações de lastlog são obtidas de um registro específico; não são uma reprodução de todas as tentativas SSH.

## 8. Rotação de logs — logrotate

Arquivos de log podem crescer muito e **esgotar o espaço em disco**. A rotação ajuda a mantê-los gerenciáveis.

**Fluxo ilustrativo:**

~~~text
app.log (arquivo ativo)
      |
      v
app.log-20261126 (arquivo antigo rotacionado)
      |
      v
Compressão / retenção / remoção conforme política
~~~

A ferramenta **logrotate** pode:
- Rotacionar por **tempo** (diariamente, semanalmente, mensalmente) ou **tamanho**;
- Renomear arquivos antigos, opcionalmente usando sufixos de data;
- Compactar logs antigos;
- Definir quantas versões manter;
- Eliminar arquivos conforme retenção;
- Adotar políticas específicas para cada serviço.

**Nota técnica:** os slides afirmam que a rotação não é ativada por padrão. Isso **depende da distribuição e do ambiente**: muitos sistemas já instalam e configuram logrotate. Não presuma que está desabilitada sem verificar. Os nomes/data de arquivos rotacionados dependem da configuração; o exemplo dos slides contém uma inconsistência entre a data escrita e a data no nome do arquivo.

Em geral, os arquivos de configuração ficam em /etc/logrotate.conf e /etc/logrotate.d/.

**Cuidados:** não apague logs manualmente em servidores sem confirmar exigências de auditoria, política de retenção, permissões e impacto nos serviços.

## 9. Relação com o Lab 245 (concluído na prática)

**O que foi confirmado no laboratório:**
1. Conexão SSH ao servidor EC2 do Vocareum.
2. Pasta de trabalho companyA verificada.
3. Leitura do arquivo de amostra /tmp/log/secure com sudo less.
4. Registros de falhas SSH, autenticação PAM e conexões FTP encontrados.
5. Saída do visualizador com q.
6. sudo lastlog executado: último login de ec2-user exibido; diversas contas de sistema apresentaram Never logged in.

Importante: o roteiro usou o arquivo de **amostra** /tmp/log/secure, diferente do caminho normalmente utilizado no sistema para os logs atuais.

[Consultar o Lab 245 — Gerenciar arquivos de log](../../laboratorios/245-gerenciar-arquivos-log-linux/README.md)

## 10. Perguntas de revisão

1. Onde os arquivos de log são normalmente armazenados no Linux?
2. Qual é o nível Syslog de maior gravidade?
3. Por que logs muito grandes representam um risco?
4. Quais comandos exibem as primeiras e as últimas dez linhas?
5. Qual comando informa o último login registrado de cada usuário?
6. Qual comando procura uma palavra dentro de um log?
7. Qual ferramenta ajuda a rotacionar e compactar logs?
8. O que faz a combinação tail -f com grep?

**Gabarito:** 1) /var/log. 2) Emergency, nível 0. 3) Podem esgotar o disco e dificultar a administração. 4) head e tail. 5) lastlog. 6) grep. 7) logrotate. 8) Acompanha novas linhas e filtra as que combinam com o padrão.

## Resumo para memorizar

- **Log = histórico de eventos** usado em investigação, segurança e operação.
- **/var/log = diretório tradicional dos logs.**
- **Syslog 0 = Emergency, mais grave; 7 = Debug, menos grave.**
- **head = início; tail = fim; tail -f = acompanhar; grep = filtrar.**
- **lastlog = último login por usuário.**
- **logrotate = controlar crescimento, compactação e retenção de logs.**

**Fonte:** slides e texto da aula Gerenciamento de arquivos de log enviados na conversa, transformados em resumo autoral. As observações técnicas de precisão foram sinalizadas separadamente. Não se trata de uma cópia integral dos slides nem de comandos adicionais executados no laboratório.
