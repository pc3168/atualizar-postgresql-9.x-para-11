# Atualizar postgresql-9.x para 11  no Debian stretch 9.6

Para usar a versão mais atualizada do software PostgreSql, precisamos adicionar o repositório postgresql.  
Rode o comando abaixo para criar um respositório.

```bash
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
Em seguida, importe a chave de assinatura do repositório 
```bash
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
```

No meu caso tive remover o repositório antigo que estava em __/etc/apt/sources.list.d/postgres94.list__
```bash
rm /etc/apt/sources.list.d/postgres94.list
```

Agora vamos atualizar o resositório 
```bash
apt-get update
```

Com o comando __dpkg-query -l postgresql*__ verificar quais versões do postgres estão instaladas.

```
postgres@teste:~$ dpkg-query -l postgresql*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Nome                                  Versão                  Arquitectura            Descrição
+++-=====================================-=======================-=======================-================================================================================
ii  postgresql-11                         11.13-1.pgdg90+1        amd64                   The World's Most Advanced Open Source Relational Database
un  postgresql-11-citus                   <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-cron                    <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-pgextwlist              <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-pglogical               <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-plsh                    <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-rum                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-wal2json                <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-9.1                        <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
ii  postgresql-9.4                        9.4.26-2.pgdg90+1       amd64                   object-relational SQL database, version 9.4 server
un  postgresql-client                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
ii  postgresql-client-11                  11.13-1.pgdg90+1        amd64                   front-end programs for PostgreSQL 11
ii  postgresql-client-9.4                 9.4.26-2.pgdg90+1       amd64                   front-end programs for PostgreSQL 9.4
ii  postgresql-client-common              226.pgdg90+1            all                     manager for multiple PostgreSQL client versions
ii  postgresql-common                     226.pgdg90+1            all                     PostgreSQL database-cluster manager
un  postgresql-contrib-11                 <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
ii  postgresql-contrib-9.4                9.4.26-2.pgdg90+1       amd64                   additional facilities for PostgreSQL
un  postgresql-doc-11                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-doc-9.4                    <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-server-dev-11              <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-server-dev-9.4             <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-server-dev-all             <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
```

Excute o comando __pg_lsclusters__ , seus clusters principais 9.4 e 11 devem estar com status online.
```
root@teste:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
9.4 main    5432 online postgres /var/lib/postgresql/9.4/main /var/log/postgresql/postgresql-9.4-mai
11  main    5433 online postgres /var/lib/postgresql/11/main  /var/log/postgresql/postgresql-11-main
```

Já existe um cluster principal para 9.4 ( uma vez que ele é criado por padrão na instalação do pacote). Isso é feito para que uma nova instalação funcione imediatamente, sem a necessidade de criar um cluster primeiro, mas pode acontecer de dar conflito quando aparecer duas opções como apareceu acima 9.4 e 11.
O procedimento recomentado é remover o cluster 11 com __pg_dropcluster__ e , em seguida atualizar com __pg_upgradecluster__.

Pare excluir o cluster 11 utilize o comando abaixo.
```bash
pg_dropcluster 11 main --stop
```

Atualize o cluster 9.4 para a versão mais recente. Esse processo pode demorar dependendo do tamanho do banco. ( No meu caso demorou 1:20 Horas) 
```bash
pg_upgradecluster 9.4 main
```

Após finalizar seu cluster 9.4 agora deve estar com status "down".

```
root@teste:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
9.4 main    5433 down   postgres /var/lib/postgresql/9.4/main /var/log/postgresql/postgresql-9.4-mai
11  main    5432 online postgres /var/lib/postgresql/11/main  /var/log/postgresql/postgresql-11-main
```

Agora, podemos remover totalmente a versão 9.4 do servidor:

```bash
apt-get --purge remove postgresql-client-9.4 postgresql-9.4
```


```
root@teste:~# dpkg-query -l postgresql*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Nome                                  Versão                  Arquitectura            Descrição
+++-=====================================-=======================-=======================-================================================================================
ii  postgresql-11                         11.13-1.pgdg90+1        amd64                   The World's Most Advanced Open Source Relational Database
un  postgresql-11-citus                   <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-cron                    <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-pgextwlist              <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-pglogical               <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-plsh                    <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-rum                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-11-wal2json                <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-9.1                        <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-client                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
ii  postgresql-client-11                  11.13-1.pgdg90+1        amd64                   front-end programs for PostgreSQL 11
ii  postgresql-client-common              226.pgdg90+1            all                     manager for multiple PostgreSQL client versions
ii  postgresql-common                     226.pgdg90+1            all                     PostgreSQL database-cluster manager
un  postgresql-contrib-11                 <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-doc-11                     <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-server-dev-11              <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
un  postgresql-server-dev-all             <nenhuma>               <nenhuma>               (nenhuma descrição disponível)
```

