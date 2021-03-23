# cdf
Instalação Opengrok
Acessar:  https://github.com/OpenGrok/OpenGrok/releases
Versão instalada 1.6.2 Opengrok
Versão atual em 5.9 Universal Ctags


OBS: Verificar Pré-requisitos para instalação da versão desejada.
•	VERSÃO DO OPENGROK DESEJADA
•	VERSÃO DO CTAGS
•	VERSÃO DO JAVA REQUERIDA
•	VERSÃO DO TOMCAT REQUERIDA

################ PREPARAÇÃO DO AMBIENTE ################
JAVA 11 ou superior
Mudar JAVA_HOME para Java requisitado
Tomcat 10 ou superior
Instalar Tomcat
curl --show-error --location https://downloads.apache.org/tomcat/tomcat-10/v10.0.2/bin/apache-tomcat-10.0.2.tar.gz | tar -xzf - --strip-components=1 -C /opt/tomcat/ 
Criar pastas e dar permissões de leitura e escrita
mkdir /opengrok/{src,data,dist,etc,log}

################ INSTALAÇÃO OPENGROK ################
BAIXAR  ctags Universal
sudo yum install universal-ctags –y 
BAIXAR E DESCOMPACTAR VERSÃO DO OPENGROK
curl --show-error --location https://github.com/oracle/opengrok/releases/download/1.6.2/opengrok-1.6.2.tar.gz | tar -xzf - --strip-components=1 -C /opengrok/dist

COPIAR ARQUIVO logging.properties 
cp /opengrok/dist/doc/logging.properties /opengrok/etc


COPIAR E RENOMEAR
Copiar o source.war que fica /opengrok/dist/lib/ renomeando para opengrok.war
 
cp /opengrok/dist/lib/source.war  /opt/tomcat/webapps/opengrok.war
###########################################################
CONFIGURAR ARQUIVO LOG
nano /opengrok/etc/logging.properties
handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

java.util.logging.FileHandler.pattern = /opengrok/log/opengrok%g.%u.log
java.util.logging.FileHandler.append = false
java.util.logging.FileHandler.limit = 0
java.util.logging.FileHandler.count = 30
java.util.logging.FileHandler.level = ALL
java.util.logging.FileHandler.formatter = org.opengrok.indexer.logger.formatter.SimpleFileLogFormatter

java.util.logging.ConsoleHandler.level = WARNING
java.util.logging.ConsoleHandler.formatter = org.opengrok.indexer.logger.formatter.SimpleFileLogFormatter

org.opengrok.level = FINE
###########################################################
CLONE DOS REPOSITÓRIOS
OBS: LOGAR COM O USUÁRIO DEFINITIVO QUE TERÁ ACESSO AO DIRETÓRIO
clone dos projetos para pasta /opengrok/src 
git init - inicializa o repositorio do git
git remote add origin https://gitlab.sefaz.pe.gov.br/GSDS/configuracao-opengrok.git - executa toda vez que for necessário reconstuir a máquina junto com a credencial.helper
git config --global credential.helper store - só será necessário uma única vez para salvar as credenciais.
git pull origin master
User e Personal Token estão no KEEPASS 

###########################################################
Ajustar o arquivo web.xml no param CONFIGURATION que fica no tomcat/webapps/opengrok/WEB-INF/
<context-param>
      <description>Full path to the configuration file where OpenGrok can read its configuration</description>
      <param-name>CONFIGURATION</param-name>
      <param-value>/opengrok/etc/configuration.xml</param-value> 
</context-param>
###########################################################
RODAR INDEXAÇÃO
java -Djava.util.logging.config.file=/opengrok/etc/logging.properties -jar /opengrok/dist/lib/opengrok.jar -c /usr/local/bin/ctags -s /opengrok/src -d /opengrok/data -H -P -S -G -W /opengrok/etc/configuration.xml -U http://localhost:8080/opengrok
###########################################################
CRIAR CRON
Obs:  Criar Script para atualização automática que execute no cron 
# toda hora das 7 às 19h de segunda a sexta
 */60 7-19 * * 1-5 /opengrok/syncOpengrok.sh
# A cada 6 horas de segunda a sexta
 */60 */6 * * 1-5   find /opengrok/log -type f -atime 0.3 -exec rm -rf {} \;
###########################################################
REMOVER PROJETOS VIA LINHA DE COMANDO
Logar com usuário opengrok senha no KEEPASS ir para diretório do opengrok
git submodule deinit <path_to_submodule> retira do .git/config
git rm --cached <path_to_submodule>
git commit -m "Removed Submodule" | rm -rf .git/modules/src/projeto

