---
layout: post
title: "Exportando e importando dados Oracle"
description: O que eu fiz com expdp e impdp
image: '/assets/img/oracle.jpg'
category: 'oracle'
---

Vou tentar mostrar como executo uma rotina para replicar dados do ambiente de produção para o ambiente de testes (maquinas diferentes) diariamente, com banco de dados oracle.

> Lembrando que o ambiente de produção esta rodando em AIX (PowerPC) e o ambiente testes está rodando OracleLinux, então estamos usando datapump.

> Outro ponto importante, é que o diretório em comum onde estão sendo gerados os backups estão mapeados via NFS

## Pra que isso?

Primeiro, tinhamos um número bem grandinho de "cacas" feitas no ambiente de produção. Entenda "cacas" como parametrizações, novos cadastros, novas contas contábeis, coisas que acabam influênciando o fluxo de trabalho de outras pessoas e tirando o sono do suporte.

Rolaram algumas reuniões e ficou definido que o ambiente ideal era o de qualidade. Problema resolvido! NAUUUUMMMMMMMM

O sistema de suporte lotou de títulos do tipo, preciso cadastrar uma nova operação e preciso testar ela em uma nota fiscal que fiz ontem. Atualiza os dados do ambiente pra mim. O atendente, na maior boa vontade, vai lá, separa algumas 5 horas do seu dia e atualiza os dados para atender o ticket e fazer mais um usuário feliz. Enquanto isso o gerente de ti é parado no corredor e recebe aquele exporro. *"Puta merda cara, você apagou a minha parametrização que fiz ontem"* e o cara nem estava sabendo do ticket maravilhoso.

Foram muitas discuções e alguns cabelos perdidos e derrepente alguem tive uma ideia maravilhosa! *"Porque não criamos um novo ambiente que atualiza todos os dias a noite, no momento que não tem mais ninguém e chamamos ele de ambiente de **Teste**"*

Beleza, todos concordaram e lá vai a parte legal. **QUEM INVENTA AGUENTA.** Configura essa biroska ai e deixa no jeito.

## Como fizemos

Moleza, download do Oracle Enterprise Linux, instalado o ambiente de Teste (Servidor ganhou até um SSD) e configurar o ambiente Oracle foi fácil. Nada que paciência, acesso a internet, google, stackoverflow não resolva.

Feito isso o plano era:

### Plano

* Criar um script no cron para rodar a atualização
* Esse script deveria apagar os dados atuais e importar os novos dados
* Esse script precisava alterar algumas informações dos sistemas para evitar aquelas coisas bestas de fazer alguma coisa em teste e jogar pra produção

### Execução

Adicionado o script ao cron

```bash
crontab -e

# Rotina de restauração de BACKUP
00 20 * * * /u02/scripts/restore/restore.sh AMBIENTE DIRETORIO
```

Agora vamos construir o script. *PS: Primeira vez que fazia isso kkkkkkkkkkkk*

Lembrando que o mesmo precisava dropar os dados anteriores e importar novos dados

```bash
#bash
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1; export ORACLE_HOME
ADMIN_SCRIPTS=/u02/scripts/admin; export ADMIN_SCRIPTS # Caminho onde tem algumas submagicas
ORACLE_SID=$1; export ORACLE_SID # Recebe o primeiro parametro do script para o ambiente
DIRECTORY=$2 # Recebe o segundo paramentro do script
DATE=$(date +%Y_%m_%d_%H%M) # Monta uma data ai
LOGIMP="IMP_${DATE}_${ORACLE_SID}.log" # Monta a string para o nome do arquivo de log

# Como não somos nós que geramos o arquivo de backup e a importação é em momento diferente, bora garimpar o nome do arquivo
cd $DIRECTORY_PATH
EXTDMP='dmp'
FILES=$(ls -ltc *.dmp | head -2 | tail -1 | tr -s ' ' | cut -d ' ' -f 9 | cut -d 'd' -f1)
DUMPFILE=$FILES$EXTDMP

# Elimina os usuários e apaga os dados
$ORACLE_HOME/bin/sqlplus / as sysdba @$ADMIN_SCRIPTS/drop_users.sql
# Roda a importação
$ORACLE_HOME/bin/impdp system/oracle dumpfile=$DUMPFILE logfile=$LOGIMP directory=$DIRECTORY exclude=STATISTICS
# Faz as atualizações marotas depois de terminar tudo
$ORACLE_HOME/bin/sqlplus / as sysdba @$ADMIN_SCRIPTS/pos_importador.sql
# Troca as senhas, pq a de prod é secreta MUAHAUHAUHAUAHAUAHAU
$ORACLE_HOME/bin/sqlplus / as sysdba @$ADMIN_SCRIPTS/user_passwords.sql
```

Beleza, agora antes de listar o script `drop_users.sql`, vou contar uma história. Sabia que não é possível eliminar usuário conectado?

É cara, o oracle não deixa. E ainda te retorna uma mensagem bem legal

```bash
ERRO na linha 1:
ORA-01940: não e possivel eliminar um usuario conectado no momento
```

E adivinha o que o datapump faz quando você não elimina um owner e manda importar ele denovo....

> TABLE_EXISTS_ACTION=[SKIP | APPEND | TRUNCATE | REPLACE]

Por acaso o default é `APPEND` e isso não ajudou muito. Moral da história, rolou um script onde antes de eliminar os owners, **matava** as sessões para evitar esse probleminha.

```sql
DECLARE
  CURSOR C_USERS IS
    SELECT USERNAME
      FROM DBA_USERS
     WHERE DEFAULT_TABLESPACE NOT IN ('SYSAUX', 'SYSTEM')
       AND USERNAME NOT IN ('XS$NULL','WEB');
BEGIN
  FOR R_USER IN C_USERS LOOP
    DBMS_OUTPUT.PUT('USUÁRIO ' || R_USER.USERNAME || ' ELIMINADO');
    FOR R IN (SELECT SID,SERIAL# FROM V$SESSION WHERE USERNAME = R_USER.USERNAME)
    LOOP
      DBMS_OUTPUT.PUT('SESSÃO ' || R.SID || ',' || R.SERIAL# || ' DO USUÁRIO ' || R_USER.USERNAME || ' ELIMINADO');
      EXECUTE IMMEDIATE 'ALTER SYSTEM KILL SESSION ''' || R.SID || ',' || R.SERIAL# || '''';
    END LOOP;
    EXECUTE IMMEDIATE 'DROP USER ' || R_USER.USERNAME || ' CASCADE';
  END LOOP;
END;
/
```

> Não me julguem por isso!!!

Legal, resolvemos o problema do drop. Agora preciso falar do que mais me tomou o tempo nessa rotina. Descobrir como iria pegar o ultimo arquivo gerado.

CARAAAAAA, isso foi muito tenso. (Eu não manjo de comandos ninjas de bash)

Então, a rotina de backup gera o arquivo dessa forma `PROD_2018_03_18_2230_18271869677.dmp`. WHAT?!

A composição do nome é AMBIENTE + DATA + HORA + FLASH_BACK_SCN + EXTENSÃO

Precisava encontar uma forma de encontrar o ultimo gerado e pegar essas informações ai para poder importar. o jeito foi esse:

```bash
EXTDMP='dmp'
FILES=$(ls -ltc *.dmp | head -2 | tail -1 | tr -s ' ' | cut -d ' ' -f 9 | cut -d 'd' -f1)
DUMPFILE=$FILES$EXTDMP
```

Agora esta todo mundo feliz (só não quando o diretorio do backup lota), pois todo dia as 20 Horas, o cron faz todo o trabalho e deixa os usuários felizes.

Ou nem tanto
```bash
O job "SYSTEM"."SYS_IMPORT_FULL_04" foi concludo com 8422 erro(s) em Seg Mar 19 22:20:25 2018 elapsed 0 02:03:01
```

É isso ae galera, isso é mais um desabafo, ou uma forma de documentar um gambiarra (ainda estou decidindo).

Obrigado pelo tempo e até o próximo post!