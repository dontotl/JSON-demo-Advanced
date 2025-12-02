# [DB] Oracle DB 23ai JSON Relational Duality HoL v2.0


Apr, 2025 



## OVERVIEW

오라클 DB 23ai 부터 JSON Relational Duality View기능이 추가되었습니다. 오라클 DB는 12c부터 JSON을 지원해왔습니다. 

19c까지 JSON 스트링을 character 데이터 타입으로 지원했고, 21c 부터 JSON을 네이티브 데이터형식으로 지원하면서 OSON이라는 바이너리 데이터 타입도 추가되었습니다. 그리고 ORDS 기반으로 MongoDB API가 제공되어서, 기존 MongoDB기반 개발 도구, 어플리케이션 코드와의 호환성이 지원되기 시작했습니다. mongosh이나 mongodb compass 등의 도구가 오라클 DB로 바로 접속이 가능하고, mongoexport, mongoimport, 개발언어의 MongoDB 접속 어댑터도 오라클DB를 대상으로 바로 접속이 가능합니다. 

오라클 DB23ai에서는 RDB 테이블의 컬럼 조합으로 JSON View를 작성할 수 있고, 작성된 JSON View를 마치 MongoDB의 collection 처럼 다룰수 있도록 발전했습니다. 

JSON Relational Duality View는 현 DB시장에서 최초로 제공되는 기능이고, 기존 NoSQL이 가지는 장점과 RDB가 가지는 장점, 더불어 NoSQL이 가지는 단점을 극복하는 기대효과를 제공할 것입니다. 

JSON 문서와 RDB테이블이 완전히 동기화된 상태로 RDB저장소 기반으로 운영됩니다. 어플리케이션에서는 JSON문서 기반의  GET/PUT/POST/DELETE 등으로 워크로드를 처리할 수 있고, 동시에 DB안에서는 SQL을 통해서도 워크로드를 처리할 수 있습니다.  JSON워크로드, SQL워크로드는 SQL로 변환되어 JSON Duality View가 바라보는 원본 테이블 데이터에 DML로 반영됩니다. 

그리고 DB 21c 부터 ORDS를 통해 AutoREST 를 지원합니다. 

AutoREST 기능을 통해 만들어진 JSON 문서에 대해 REST API를 활성화하면, 자동으로 GET/POST/PUT/DELETE 등의 API함수가 만들어져서 제공되며 Swagger 기반의 API 스펙문서까지 자동 생성되어 제공됩니다. 

JSON Relational Duality View의 가장 좋은 점은 JSON 데이터의 단점일수 있는 중복 데이터 관리 오버헤드를  RDB의 정규화된 설계를 통해 제거한다는 것입니다. 저장소를 RDB로 정규화하고 최적화하여 중복 데이터없이 관리할 수 있고, NoSQL의 한계인 고급 분석을 위해 다양한 방법을 적용할 필요없이, 오라클 DB안에서 실시간 고급  SQL분석이 가능하다는 점입니다. 

실습을 통해 JSON Relational Duality View을 테스트해 보고, JSON 기반 어플리케이션에서의 활용이나  rest API 의 백엔드 저장소로서의 활용,  noSQL 페인포인트 해결의 아이디어를 얻는 시간이 되기를 바랍니다. 

## **실습 환경 구성**

간단한 실습 환경 구현을 위해 db23ai free docker 버전을 사용하겠습니다. 도커환경을 윈도우나 Mac에서 사전구성한 후 진행합니다.

본 데모는 M1 Mac 환경에서 테스트하였습니다. 

### DB23ai 도커버전 배포

```jsx
demo@local ~ % docker run -d --name db23ai -p 1521:1521 -p 8080:8080 -p 27017:27017 -e ORACLE_PWD=Welcome_12345 -e ENABLE_ARCHIVELOG=true container-registry.oracle.com/database/free:23.7.0.0-arm64
Unable to find image 'container-registry.oracle.com/database/free:23.7.0.0-arm64' locally
23.7.0.0-arm64: Pulling from database/free
31c43fb56fa8: Pull complete 
bc317eaf942e: Pull complete 
438466fd0c88: Pull complete 
7dc0d92c9cd9: Pull complete 
d54f43c3fbe5: Pull complete 
6974857549ef: Pull complete 
67ba843ee1a9: Pull complete 
4686a5b0a300: Pull complete 
128efe52bde4: Pull complete 
ba346bf7c629: Pull complete 
19ed560d8027: Pull complete 
ef2685c1d74b: Pull complete 
ab63d79483cb: Pull complete 
7272e5dfb954: Pull complete 
ff68f6874860: Pull complete 
fed61e7db61e: Pull complete 
ee305391b24c: Pull complete 
562523b05053: Pull complete 
691f051927dd: Pull complete 
27ec8b700b12: Pull complete 
717171bac790: Pull complete 
4b6b25d1a5dd: Pull complete 
Digest: sha256:d1462e24c082c8584c899e44b436a3ee7259d7e78fd2f9502bafe5b6e826299b
Status: Downloaded newer image for container-registry.oracle.com/database/free:23.7.0.0-arm64
2c4cd29c7a5a3cea2d91df5f60245a16624c414eacd79029e30369e7912f25fd

demo@local ~ % docker ps
CONTAINER ID   IMAGE                                                             COMMAND                   CREATED          STATUS                             PORTS                                       NAMES
ac396ab8b0bf   container-registry.oracle.com/database/free:23.7.0.0-lite-arm64   "/bin/bash -c $ORACL…"   24 seconds ago   Up 23 seconds (health: starting)   0.0.0.0:1521->1521/tcp, :::1521->1521/tcp   db23ai

demo@local ~ % docker logs db23ai
Expanding oracle data
/home/oracle
Starting Oracle Net Listener.
Oracle Net Listener started.
Starting Oracle Database instance FREE.
Oracle Database instance FREE started.
Pluggable Database FREEPDB1 opened.

SQL*Plus: Release 23.0.0.0.0 - Production on Wed Apr 2 07:59:00 2025
Version 23.7.0.25.01

Copyright (c) 1982, 2025, Oracle.  All rights reserved.

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.7.0.25.01

SQL> 
User altered.

SQL> 
User altered.

SQL> 
Session altered.

SQL> 
User altered.

SQL> Disconnected from Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.7.0.25.01
#########################
DATABASE IS READY TO USE!
#########################
The following output is now a tail of the alert.log:
Pluggable database FREEPDB1 with pdb id - 3 is now marked as NEW.
****************************************************************
2025-04-02T07:58:56.896177+00:00
FREEPDB1(3):Opening pdb with Resource Manager plan: DEFAULT_PLAN
Completed: Pluggable database FREEPDB1 opened read write 
Completed:             alter pluggable database all open
            alter pluggable database FREEPDB1 save state
Completed:             alter pluggable database FREEPDB1 save state
2025-04-02T07:59:00.602248+00:00
FREEPDB1(3):TABLE AUDSYS.AUD$UNIFIED: ADDED INTERVAL PARTITION SYS_P303 (3929) VALUES LESS THAN (TIMESTAMP' 2025-04-03 00:00:00')
2025-04-02T07:59:43.332557+00:00
TABLE SYS.WRP$_REPORTS: ADDED AUTOLIST FRAGMENT SYS_P373 (2) VALUES (( 1466292468, TO_DATE(' 2025-03-31 00:00:00', 'syyyy-mm-dd hh24:mi:ss', 'nls_calendar=gregorian') ))
TABLE SYS.WRP$_REPORTS_DETAILS: ADDED AUTOLIST FRAGMENT SYS_P374 (2) VALUES (( 1466292468, TO_DATE(' 2025-03-31 00:00:00', 'syyyy-mm-dd hh24:mi:ss', 'nls_calendar=gregorian') ))
TABLE SYS.WRP$_REPORTS_TIME_BANDS: ADDED AUTOLIST FRAGMENT SYS_P377 (2) VALUES (( 1466292468, TO_DATE(' 2025-03-31 00:00:00', 'syyyy-mm-dd hh24:mi:ss', 'nls_calendar=gregorian') ))
2025-04-02T08:02:04.099250+00:00
FREEPDB1(3):Resize operation completed for file# 17, fname /opt/oracle/oradata/FREE/FREEPDB1/sysaux01.dbf, old size 409600K, new size 419840K
2025-04-02T08:02:04.115915+00:00
Resize operation completed for file# 3, fname /opt/oracle/oradata/FREE/sysaux01.dbf, old size 624640K, new size 645120K
```

M1 계열은 ARM기반 아키텍처라서, Arm 기반 Oracle DB23ai 를 배포했습니다. 

x86 계열 테스트 환경은 x86_64 기반 Oracle DB23ai 를 배포하면 되겠습니다. 
 "DATABASE IS READY TO USE!" 문구가 로그에 뜨면 DB접속에 접속하고 사용 가능한 상태가 됩니다. 

### ORDS 설치 및 구성

도커 컨테이너에 접속해 dnf update를 수행합니다. 

```jsx
demo@local ~ % docker exec -it db23ai bash
bash-4.4$ whoami
oracle

bash-4.4$ su - 

[root@2c4cd29c7a5a etc]# dnf update -y
```

dnf가 타임아웃 에러가 나면, 아래 워크어라운드를 적용한 후 다시 update 해 줍니다

참조 URL : https://github.com/oracle/docker-images/issues/2900

```jsx
DNF를 쓰는 경우 

# mv /etc/dnf/vars/ociregion /etc/dnf/vars/ociregion.old
# echo -n "" > /etc/dnf/vars/ociregion

YUM을 쓰는 경우 
# mv /etc/yum/vars/ociregion /etc/yum/vars/ociregion.old
# echo -n "" > /etc/yum/vars/ociregion
```

DB 접속을 테스트해 봅니다

```jsx
[root@2c4cd29c7a5a etc]# exit
logout

bash-4.4$ sqlplus sys/Welcome_12345@localhost:1521/FREEPDB1 as sysdba

SQL*Plus: Release 23.0.0.0.0 - Production on Wed Apr 2 08:04:05 2025
Version 23.7.0.25.01

Copyright (c) 1982, 2025, Oracle.  All rights reserved.

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
Version 23.7.0.25.01

SQL>
 
```

ORDS를 dnf를 사용하여 설치하도록 하겠습니다. 

```jsx
[root@d725788815fb ~]# dnf install oracle-software-release-el8.aarch64

[root@d725788815fb ~]# dnf search ords
Oracle Software for Oracle Linux 8 (aarch64)                                     16 kB/s |  50 kB     00:03    
Last metadata expiration check: 0:00:01 ago on Tue Apr  8 02:25:55 2025.
========================================== Name Exactly Matched: ords ==========================================
ords.noarch : Oracle REST Data Services

[root@d725788815fb ~]# dnf install ords -y
```

ORDS 구성을 위해서는 우선 JAVA 11 (ORDS 24 버전은 JAVA 17이상) 버전이상이 필요합니다. 

JAVA 환경이 export 안되어 있다면bash_profile 환경변수 셋업을 해주고 진행합니다. 

ords config install 를 실행하여, ORDS 구성을 진행합니다. 

/etc/ords/confg

ords --config /etc/ords/config install

```jsx
[root@d725788815fb ~]# dnf install java-17 -y

[root@d725788815fb ~]# exit
logout

bash-4.4$ echo -e 'export PATH="$PATH:/home/oracle/ords/bin:/opt/oracle/product/23ai/dbhomeFree/jdk/bin"' >> ~/.bash_profile

bash-4.4$ . ~/.bash_profile

bash-4.4$ ords --config /opt/oracle/ords/config install

ORDS: Release 24.4 Production on Thu Apr 03 07:51:21 2025

Copyright (c) 2010, 2025, Oracle.

Configuration:
  /opt/oracle/ords/config

The configuration folder /opt/oracle/ords/config does not contain any configuration files.

Oracle REST Data Services - Interactive Install

  Enter a number to select the TNS net service name to use from /opt/oracle/product/23ai/dbhomeFree/network/admin/tnsnames.ora or specify the database connection
    [1] FREE         SERVICE_NAME=FREE                                           
    [2] FREEPDB1     SERVICE_NAME=FREEPDB1                                       
    [S] Specify the database connection
  Choose [1]: 2
  Provide database user name with administrator privileges.
    Enter the administrator username: sys
  Enter the database password for SYS AS SYSDBA: 

Retrieving information.
ORDS is not installed in the database. ORDS installation is required.

  Enter a number to update the value or select option A to Accept and Continue
    [1] Connection Type: TNS
    [2] TNS Connection: TNS_NAME=FREEPDB1 TNS_FOLDER=/opt/oracle/product/23ai/dbhomeFree/network/admin
           Administrator User: SYS AS SYSDBA
    [3] Database password for ORDS runtime user (ORDS_PUBLIC_USER): <generate>
    [4] ORDS runtime user and schema tablespaces:  Default: SYSAUX Temporary TEMP
    [5] Additional Feature: Database Actions
    [6] Configure and start ORDS in Standalone Mode: Yes
    [7]    Protocol: HTTP
    [8]       HTTP Port: 8080
    [A] Accept and Continue - Create configuration and Install ORDS in the database
    [Q] Quit - Do not proceed. No changes
  Choose [A]: A
```

구성이 진행되면 ORDS가 기동되어 터미널에서 로그를 볼 수 있습니다

```jsx
The setting named: db.connectionType was set to: tns in configuration: default
The setting named: db.tnsAliasName was set to: FREEPDB1 in configuration: default
The setting named: db.tnsDirectory was set to: /opt/oracle/product/23ai/dbhomeFree/network/admin in configuration: default
The setting named: db.username was set to: ORDS_PUBLIC_USER in configuration: default
The setting named: db.password was set to: ****** in configuration: default
The setting named: feature.sdw was set to: true in configuration: default
The global setting named: database.api.enabled was set to: true
The setting named: restEnabledSql.active was set to: true in configuration: default
The global setting named: standalone.http.port was set to: 8080
The global setting named: standalone.doc.root was set to: /opt/oracle/ords/config/global/doc_root
The setting named: security.requestValidationFunction was set to: ords_util.authorize_plsql_gateway in configuration: default
2025-04-03T07:51:58.005Z INFO        Created folder /home/oracle/ords/logs
2025-04-03T07:51:58.007Z INFO        The log file is defaulted to the current working directory located at /home/oracle/ords/logs
2025-04-03T07:51:58.140Z INFO        Installing Oracle REST Data Services version 24.4.0.r3451601 in FREEPDB1
2025-04-03T07:51:59.143Z INFO        ... Verified database prerequisites
2025-04-03T07:51:59.470Z INFO        ... Created Oracle REST Data Services proxy user
2025-04-03T07:51:59.725Z INFO        ... Created Oracle REST Data Services schema
2025-04-03T07:52:00.146Z INFO        ... Granted privileges to Oracle REST Data Services
2025-04-03T07:52:03.175Z INFO        ... Created Oracle REST Data Services database objects
2025-04-03T07:52:11.738Z INFO        Completed installation for Oracle REST Data Services version 24.4.0.r3451601. Elapsed time: 00:00:13.579 

2025-04-03T07:52:11.742Z INFO        Log file written to /home/oracle/ords/logs/ords_install_2025-04-03_075158_00738.log
2025-04-03T07:52:11.892Z INFO        HTTP and HTTP/2 cleartext listening on host: 0.0.0.0 port: 8080
2025-04-03T07:52:11.907Z INFO        Disabling document root because the specified folder does not exist: /opt/oracle/ords/config/global/doc_root
2025-04-03T07:52:11.907Z INFO        Default forwarding from / to contextRoot configured.
2025-04-03T07:52:13.348Z INFO        Configuration properties for: |default|lo|
db.password=******
db.tnsAliasName=FREEPDB1
conf.use.wallet=true
security.requestValidationFunction=ords_util.authorize_plsql_gateway
database.api.enabled=true
db.username=ORDS_PUBLIC_USER
standalone.http.port=8080
restEnabledSql.active=true
resource.templates.enabled=false
feature.sdw=true
config.required=true
db.connectionType=tns
standalone.doc.root=/opt/oracle/ords/config/global/doc_root
db.tnsDirectory=/opt/oracle/product/23ai/dbhomeFree/network/admin

2025-04-03T07:52:13.349Z WARNING     *** jdbc.MaxLimit in configuration |default|lo| is using a value of 10, this setting may not be sized adequately for a production environment ***
2025-04-03T07:52:13.522Z INFO        

Mapped local pools from /opt/oracle/ords/config/databases:
  /ords/                              => default                        => VALID     

2025-04-03T07:52:13.554Z INFO        Oracle REST Data Services initialized
Oracle REST Data Services version : 24.4.0.r3451601
Oracle REST Data Services server info: jetty/12.0.13
Oracle REST Data Services java info: OpenJDK 64-Bit Server VM (Red_Hat-17.0.14.0.7-3.0.1) (build: 17.0.14+7-LTS mixed mode, sharing)
```

ctrl+c 를 눌러 쉘로 빠져나온후, Mongo DB 호환 API를 활성화하고 재기동합니다. 

```sql
bash-4.4$ ords config set mongo.enabled true

bash-4.4$ ords serve
```

### 샘플 스키마 구성

SQL*Developer with Visual Studio Code

링크 : [https://marketplace.visualstudio.com/items?itemName=Oracle.sql-developer](https://marketplace.visualstudio.com/items?itemName=Oracle.sql-developer)

개발도구 VS Code에서 사용할 수 있는 SQLDeveloper가 익스텐션으로 제공됩니다.

무료로 사용가능하며, 개발자 선호하는 IDE환경에서별도 도구없이 효과적으로 DB 접속과 어플리케이션 개발이 가능합니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image.png)

VSCode 버전 SQLDevleoper나, Docker 내부 sqlplus를 이용해서 DB에 sys 유저로 접속하여 아래 SQL스크립트를 반영합니다. 

```coq
-- 유저 생성
create user jsondual identified by jsondual default tablespace users quota unlimited on users;
grant connect, resource to jsondual;

-- 샘플 테이블, DV, JSON 데이터 구성 
DROP VIEW IF EXISTS team_dv;
DROP VIEW IF EXISTS race_dv;
DROP VIEW IF EXISTS driver_dv;
DROP TABLE IF EXISTS driver_race_map;
DROP TABLE IF EXISTS race;
DROP TABLE IF EXISTS driver;
DROP TABLE IF EXISTS team;

CREATE TABLE IF NOT EXISTS team
(team_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
name    VARCHAR2(255) NOT NULL UNIQUE,
points  INTEGER NOT NULL,
CONSTRAINT team_pk PRIMARY KEY(team_id));

CREATE TABLE IF NOT EXISTS driver
(driver_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
name      VARCHAR2(255) NOT NULL UNIQUE,
points    INTEGER NOT NULL,
team_id   INTEGER,
CONSTRAINT driver_pk PRIMARY KEY(driver_id),
CONSTRAINT driver_fk FOREIGN KEY(team_id) REFERENCES team(team_id));

CREATE TABLE IF NOT EXISTS race
(race_id   INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
name      VARCHAR2(255) NOT NULL UNIQUE,
laps      INTEGER NOT NULL,
race_date DATE,
podium  JSON,
CONSTRAINT   race_pk PRIMARY KEY(race_id));

CREATE TABLE IF NOT EXISTS driver_race_map
(driver_race_map_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
race_id            INTEGER NOT NULL,
driver_id          INTEGER NOT NULL,
position           INTEGER,
CONSTRAINT driver_race_map_pk  PRIMARY KEY(driver_race_map_id),
CONSTRAINT driver_race_map_fk1 FOREIGN KEY(race_id) REFERENCES race(race_id),
CONSTRAINT driver_race_map_fk2 FOREIGN KEY(driver_id) REFERENCES driver(driver_id));

CREATE OR REPLACE TRIGGER driver_race_map_trigger
BEFORE INSERT ON driver_race_map
FOR EACH ROW
DECLARE
    v_points  INTEGER;
    v_team_id INTEGER;
BEGIN
SELECT team_id INTO v_team_id FROM driver WHERE driver_id = :NEW.driver_id;
IF :NEW.position = 1 THEN
    v_points := 25;
ELSIF :NEW.position = 2 THEN
    v_points := 18;
ELSIF :NEW.position = 3 THEN
    v_points := 15;
ELSIF :NEW.position = 4 THEN
    v_points := 12;
ELSIF :NEW.position = 5 THEN
    v_points := 10;
ELSIF :NEW.position = 6 THEN
    v_points := 8;
ELSIF :NEW.position = 7 THEN
    v_points := 6;
ELSIF :NEW.position = 8 THEN
    v_points := 4;
ELSIF :NEW.position = 9 THEN
    v_points := 2;
ELSIF :NEW.position = 10 THEN
    v_points := 1;
ELSE
    v_points := 0;
END IF;
UPDATE driver SET points = points + v_points
    WHERE driver_id = :NEW.driver_id;
UPDATE team SET points = points + v_points
    WHERE team_id = v_team_id;
END;
/

CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW race_dv AS
SELECT JSON {
            '_id' IS r.race_id,
            'name'   IS r.name,
            'laps'   IS r.laps WITH NOUPDATE,
            'date'   IS r.race_date,
            'podium' IS r.podium WITH NOCHECK,
            'result' IS
                [ SELECT JSON {'driverRaceMapId' IS drm.driver_race_map_id,
                                'position'        IS drm.position,
                                UNNEST
                                (SELECT JSON {'driverId' IS d.driver_id,
                                                'name'     IS d.name}
                                    FROM driver d WITH NOINSERT UPDATE NODELETE
                                    WHERE d.driver_id = drm.driver_id)}
                    FROM driver_race_map drm WITH INSERT UPDATE DELETE
                    WHERE drm.race_id = r.race_id ]}
    FROM race r WITH INSERT UPDATE DELETE;
    
CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW driver_dv AS
SELECT JSON {'_id' IS d.driver_id,
        'name'     IS d.name,
        'points'   IS d.points,
        UNNEST
            (SELECT JSON {'teamId' IS t.team_id,
                        'team'   IS t.name WITH NOCHECK}
                FROM team t WITH NOINSERT NOUPDATE NODELETE
                WHERE t.team_id = d.team_id),
        'race'     IS
            [ SELECT JSON {'driverRaceMapId' IS drm.driver_race_map_id,
                            UNNEST
                            (SELECT JSON {'raceId' IS r.race_id,
                                            'name'   IS r.name}
                                FROM race r WITH NOINSERT NOUPDATE NODELETE
                                WHERE r.race_id = drm.race_id),
                            'finalPosition'   IS drm.position}
                FROM driver_race_map drm WITH INSERT UPDATE NODELETE
                WHERE drm.driver_id = d.driver_id ]}
FROM driver d WITH INSERT UPDATE DELETE;

CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW team_dv AS
SELECT JSON {'_id'  IS t.team_id,
            'name'    IS t.name,
            'points'  IS t.points,
            'driver'  IS
                [ SELECT JSON {'driverId' IS d.driver_id,
                                'name'     IS d.name,
                                'points'   IS d.points WITH NOCHECK}
                    FROM driver d WITH INSERT UPDATE
                    WHERE d.team_id = t.team_id ]}
    FROM team t WITH INSERT UPDATE DELETE;
    
INSERT INTO team_dv VALUES ('{"_id" : 301,
                        "name"   : "Red Bull",
                        "points" : 0,
                        "driver" : [ {"driverId" : 101,
                                        "name"     : "Max Verstappen",
                                        "points"   : 0},
                                    {"driverId" : 102,
                                        "name"     : "Sergio Perez",
                                        "points"   : 0} ]}');

INSERT INTO team_dv VALUES ('{"_id" : 302,
                            "name"   : "Ferrari",
                            "points" : 0,
                            "driver" : [ {"driverId" : 103,
                                            "name"     : "Charles Leclerc",
                                            "points"   : 0},
                                        {"driverId" : 104,
                                            "name"     : "Carlos Sainz Jr",
                                            "points"   : 0} ]}');

INSERT INTO team_dv VALUES ('{"_id" : 2,
                            "name"   : "Mercedes",
                            "points" : 0,
                            "driver" : [ {"driverId" : 105,
                                            "name"     : "George Russell",
                                            "points"   : 0},
                                        {"driverId" : 106,
                                            "name"     : "Lewis Hamilton",
                                            "points"   : 0} ]}');
COMMIT;    

INSERT INTO race_dv VALUES ('{"_id" : 201,
                            "name"   : "Bahrain Grand Prix",
                            "laps"   : 57,
                            "date"   : "2022-03-20T00:00:00",
                            "podium" : {}}');

INSERT INTO race_dv VALUES ('{"_id" : 202,
                            "name"   : "Saudi Arabian Grand Prix",
                            "laps"   : 50,
                            "date"   : "2022-03-27T00:00:00",
                            "podium" : {}}');

INSERT INTO race_dv VALUES ('{"_id" : 203,
                            "name"   : "Australian Grand Prix",
                            "laps"   : 58,
                            "date"   : "2022-04-09T00:00:00",
                            "podium" : {}}');
COMMIT;

```

DB에 jsondual 유저로 접속하여 다음 SQL을 실행하여 ORDS서비스를 활성화합니다

```jsx
execute ords.ENABLE_SCHEMA;
```

docker exec -it db23ai bash

sqlplus sys/Welcome_12345@localhost:1521/FREEPDB1 as sysdba
create user jsondual identified by jsondual default tablespace users quota unlimited on users;
grant connect, resource to jsondual;

conn jsondual/jsondual@localhost:1521/FREEPDB1

execute ords.ENABLE_SCHEMA;

```coq
begin
	ords.enable_schema(
			p_enabled=>TRUE,
			p_schema=>'JSONDUAL',
			p_url_mapping_type=>'BASE_PATH',
			p_url_mapping_pattern=>'jd',
```

## JSON Duality View 데모

이제 웹브라우저에서 ords랜딩 페이지에 접속해 봅니다

[http://localhost:8080/ords/_/landing](http://localhost:8080/ords/_/landing)

ORDS가 제공하는 서비스 소개 페이지가 보입니다. 

본 데모에서는 APEX는 구성하지 않았고, 주로 SQL Developer Web를 사용할 것입니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%201.png)

SQL Developer Web 의 Go버튼을 클릭합니다 

로그인페이지에서 jsondual/jsondual 로 로그인하여 DB Actions 런치패드로 진입합니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/e3359d7a-d7ee-4c7d-816d-b9cb02b9360e.png)

DB Actions는 ORDS기반으로 DB개발과 관리를 제공하는 여러 도구들을 웹서비스 기반으로 제공하는 서비스입니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%202.png)

개발 탭에서 JSON기반 앱 개발을 돕는 SQLDeveloper 웹 버전,  REST 서비스 생성, JSON  도구 등을 활용할 수 있습니다

왼쪽 메뉴의  SQL을 선택하고 화면 우측 하단의 Open을 선택합니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%203.png)

sql*developer 웹 버전이 실행되었습니다.

이미 생성한 DRIVER, RACE, TEAM, DRIVER_RACE_MAP 테이블 들이 보입니다. 

클릭하여 구조나 데이터를 조회해 보면 일반적인 RDB테이블임을 알 수 있습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%204.png)

Navigator에서 필터를 Tables 에서 Views 로 전환하면, 생성된 DRIVER_DV, RACE_DV, TEAM_DV가 보입니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%205.png)

각 View를 오른클릭하면 메뉴가 나오는데, 여기서 REST > Enable을 선택합니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%206.png)

팝업창에서 Enable을 선택하면, DRIVER_DV에 대해 AutoRest가 활성화 되어, 해당 View 데이터를 rest api로 서비스하게 됩니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%207.png)

나머지 RACE_DV, TEAM_DV에도 동일하게 REST 를 Enable 해 줍니다.

테이블 컬럼이 조합된 뷰를 선언하여 JSON Duality View를 만들고, 추가 코딩이나 개발없이 간단하게 REST 서비스를 활성화 하여

 손쉽게 오라클 DB기반의 rest API 서비스를 제공할 수 있게 되는 것입니다.  

이제 왼쪽 상단의 햄버거 메뉴를 선택해서 REST 메뉴를 선택합니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%208.png)

이전에 Enable 한 REST api 가 구성되어 있고 이를 관리하는 웹 콘솔로 연계가 됩니다. 

AUTOREST에 3개의 REST API가 활성화 되었다고 표기되고 있습니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%209.png)

간단하게 브라우저에서 json 데이터를 get 해 보겠습니다.
AUTOREST 를 선택합니다.

3개의 JSON Duality View가 보이는데, DRIVER_DV의 rest api를 call 해 보겠습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2010.png)

화면에 http://<ip>:8080/ords/freepdb1/jsondual/driver_dv/ 라는 서비스 엔드포인트 주소가 보입니다. 

그 오른쪽에 팝업버튼이 있는데 이를 선택합니다. 

driver_dv의 데이터를 get한 결과를 볼 수 있고, pretty print 적용을 활성화하면 정돈된 JSON 데이터를 볼수 있습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2011.png)

### MongoDB API 호환 실습 데모

mongodb 도구 구성을 위한 리파지토리를 추가해 줍니다 

```sql
[root@d725788815fb ~]# vi /etc/yum.repos.d/mongodb-org-8.0.repo

[mongodb-org-8.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/8.0/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-8.0.asc
```

리파지토리 없이 다음 경로에서 적절한 아키텍처 버전의 도구를 찾아서 바로 설치도 가능합니다 

참조 URL : https://repo.mongodb.org/yum/redhat/8/mongodb-org/

dnf install -y [https://repo.mongodb.org/yum/redhat/8/mongodb-org/8.0/aarch64/RPMS/mongodb-mongosh-2.5.0.aarch64.rpm](https://repo.mongodb.org/yum/redhat/8/mongodb-org/8.0/aarch64/RPMS/mongodb-mongosh-2.5.0.aarch64.rpm)

dnf install -y [https://repo.mongodb.org/yum/redhat/8/mongodb-org/8.0/aarch64/RPMS/mongodb-database-tools-100.12.0-1.aarch64.rpm](https://repo.mongodb.org/yum/redhat/8/mongodb-org/8.0/aarch64/RPMS/mongodb-database-tools-100.12.0-1.aarch64.rpm)

mongodb 관련도구를 설치합니다 

```jsx
[root@d725788815fb ~]# dnf install -y mongodb-mongosh
 
[root@d725788815fb ~]# dnf install -y mongodb-database-tools
```

이제 오라클 DB의 MongoDB API호환기능을 사용해, 몽고DB처럼 접속해 보겠습니다.

MongoDB의 커넥션 스트링은 다음과 같습니다. 

```sql
export URI='mongodb://<user>:<password>@<host>:27017/<user>?authMechanism=PLAIN&authSource=$external&tls=true&retryWrites=false&loadBalanced=true'
```

테스트 환경에 맞춘 URI는 다음과 같습니다. 접속을 수행해 보겠습니다. 

```sql
export URI='mongodb://jsondual:jsondual@localhost:27017/jsondual?authMechanism=PLAIN&authSource=$external&tls=true&retryWrites=false&loadBalanced=true' 
mongosh --tlsAllowInvalidCertificates $URI
```

오라클 DB의 호환 API기능으로 마치 mongodb 처럼 클라이언트가 접속됩니다.

```jsx
bash-4.4$ export URI='mongodb://jsondual:jsondual@localhost:27017/jsondual?authMechanism=PLAIN&authSource=$extnal&tls=true&retryWrites=false&loadBalanced=true' 
bash-4.4$ mongosh --tlsAllowInvalidCertificates $URI
Current Mongosh Log ID:	67f5c96a7e6ccd9e9965d0fa
Connecting to:		mongodb://<credentials>@localhost:27017/jsondual?authMechanism=PLAIN&authSource=%24external&tls=true&retryWrites=false&loadBalanced=true&serverSelectionTimeoutMS=2000&tlsAllowInvalidCertificates=true&appName=mongosh+2.5.0
Using MongoDB:		4.2.14
Using Mongosh:		2.5.0

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

jsondual>
```

mongodb 콜렉션을 조회하면 oracle에서 만든 DV들이 잘 보입니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2012.png)

show collections 

mongosh에서 데이터를 조회해 봅니다.

```jsx
jsondual> db.driver_dv.countDocuments()
6
jsondual> db.driver_dv.find()
[
  {
    _id: 101,
    name: 'Max Verstappen',
    points: 0,
    teamId: 301,
    team: 'Red Bull',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('+dmBXf8nh59hOGz9FiKwZQ==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  },
  {
    _id: 102,
    name: 'Sergio Perez',
    points: 0,
    teamId: 301,
    team: 'Red Bull',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('mGWine5fV1RnSpsdLsWHMA==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  },
  {
    _id: 103,
    name: 'Charles Leclerc',
    points: 0,
    teamId: 302,
    team: 'Ferrari',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('xHqR9Xvzz0Xw2ChCQDmakA==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  },
  {
    _id: 104,
    name: 'Carlos Sainz Jr',
    points: 0,
    teamId: 302,
    team: 'Ferrari',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('Nj7Gu/D63ZE7IZSClZ2jnQ==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  },
  {
    _id: 105,
    name: 'George Russell',
    points: 0,
    teamId: 2,
    team: 'Mercedes',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('qLsYJfYhjsDTAGcRc1QFlw==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  },
  {
    _id: 106,
    name: 'Lewis Hamilton',
    points: 0,
    teamId: 2,
    team: 'Mercedes',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('0/8yE3k+MGIEu15QYDaOQQ==', 0),
      asof: Binary.createFromBase64('AAAAAAA0p1k=', 0)
    }
  }
]
```

데이터를 업데이트 해봅니다.

아래 처럼 드라이버 101 의 이름을 바꿔보도록 하겠습니다. 

```jsx
db.driver_dv.updateOne({ "_id": 101 }, { $set: { "name": "JSON Dual view working with Mongo API" } })
```

업데이트 후 조회하면, 잘 반영된 것을 확인할 수 있습니다. 

```jsx
jsondual> db.driver_dv.updateOne({ "_id": 101 }, { $set: { "name": "JSON Dual view working with Mongo API" } })
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
jsondual> db.driver_dv.find()
[
  {
    _id: 101,
    name: 'JSON Dual view working with Mongo API',
    points: 0,
    teamId: 301,
    team: 'Red Bull',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('x8OwucFV6a+HZ+S+ZZtBbw==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  },
  {
    _id: 102,
    name: 'Sergio Perez',
    points: 0,
    teamId: 301,
    team: 'Red Bull',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('mGWine5fV1RnSpsdLsWHMA==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  },
  {
    _id: 103,
    name: 'Charles Leclerc',
    points: 0,
    teamId: 302,
    team: 'Ferrari',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('xHqR9Xvzz0Xw2ChCQDmakA==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  },
  {
    _id: 104,
    name: 'Carlos Sainz Jr',
    points: 0,
    teamId: 302,
    team: 'Ferrari',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('Nj7Gu/D63ZE7IZSClZ2jnQ==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  },
  {
    _id: 105,
    name: 'George Russell',
    points: 0,
    teamId: 2,
    team: 'Mercedes',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('qLsYJfYhjsDTAGcRc1QFlw==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  },
  {
    _id: 106,
    name: 'Lewis Hamilton',
    points: 0,
    teamId: 2,
    team: 'Mercedes',
    race: [],
    _metadata: {
      etag: Binary.createFromBase64('0/8yE3k+MGIEu15QYDaOQQ==', 0),
      asof: Binary.createFromBase64('AAAAAAA0qyE=', 0)
    }
  }
]
```

오라클 DB에 잘 반영되었는지 확인해 보겠습니다. 

AutoREST 관리화면에서 driver_dv 의 url을 브라우저에서 call 해 봅니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2013.png)

반영된 데이터가 API call로 잘 보입니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2014.png)

 SQL*Developer 에서 테이블로 데이터를 확인하면 다음과 같습니다.

변경된 데이터가 driver 테이블에 잘 반영되어 있습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2015.png)

JSON Duality View를 확인하면 다음과 같습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2016.png)

### MongoDB Compass 연계

MongoDB Compass로도 오라클DB JSON DV접속해 보도록 하겠습니다. 

mongosh과 같은 URI를 적용하면 됩니다.

우선 mongodb compass를 다운로드 합니다. 

URL : [https://www.mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass)  

실행 후 add new connection을 선택합니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2017.png)

URI에 다음 오라클DB 접속정보를 넣습니다

```jsx
mongodb://jsondual:jsondual@localhost:27017/jsondual?authMechanism=PLAIN&authSource=$extnal&tls=true&retryWrites=false&loadBalanced=true
```

URI 에 커넥션 스트링을 붙여넣고 name은 db23ai로 입력합니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2018.png)

한가지 추가로 셋업할 것이 있는데, 

Advanced Connection Options > TLS/SSL 에서 tlsAllowInvalidCertificates를 활성화하고 Save & Connect 버튼을 선택합니다

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2019.png)

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2020.png)

접속이 되면, compass를 통해 DV데이터를 mongodb 자체 콜렉션처럼 관리할 수 있습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2021.png)

### 별첨

1. **Autorest 활성화 SQL Script** 

데모에서는 SQL*Developer Web으로 Autorest를 활성화 했는데, JSON DV의 Autorest 를 활성화하는 SQL스크립트는 다음과 같습니다. 

```jsx
begin
ORDS.ENABLE_OBJECT(
    P_ENABLED        => TRUE,
    P_SCHEMA         => 'JSONDUAL',
    P_OBJECT         =>  'DRIVER_DV',
    P_OBJECT_TYPE    => 'VIEW',
    P_OBJECT_ALIAS   => 'driver_dv',
    P_AUTO_REST_AUTH => FALSE
);
COMMIT;

ORDS.ENABLE_OBJECT(
    P_ENABLED        => TRUE,
    P_SCHEMA         => 'JSONDUAL',
    P_OBJECT         =>  'RACE_DV',
    P_OBJECT_TYPE    => 'VIEW',
    P_OBJECT_ALIAS   => 'race_dv',
    P_AUTO_REST_AUTH => FALSE
);
COMMIT;

ORDS.ENABLE_OBJECT(
    P_ENABLED        => TRUE,
    P_SCHEMA         => 'JSONDUAL',
    P_OBJECT         =>  'TEAM_DV',
    P_OBJECT_TYPE    => 'VIEW',
    P_OBJECT_ALIAS   => 'team_dv',
    P_AUTO_REST_AUTH => FALSE
);
COMMIT;
end;
/
```

1. **Oracle Live SQL for JSON Duality View**

JSON Duality View를 간단하게 체험할 수 있는 LiveSQL 주소입니다.

웹 브라우저와 [oracle.com](http://oracle.com) 아이디만 있으면 실습이 가능합니다

https://livesql.oracle.com/next/worksheet?tutorial=json-duality-views-quick-start-D3wdHG&share_key=jCX1875rL3

1. **JSON Duality View Builder - SQL*Developer for VS Code**

SQL Developer for VS code 에서 Duality View Builder 를 제공합니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2022.png)

Name을 입력하고 기반이 되는 Add Root Table 를 선택해서 테이블을 추가합니다. 

Driver 테이블을 기반으로 선택합니다.

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2023.png)

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2024.png)

Driver 테이블이 보이고 하단에 기본 JSON DV 스크립트가 나옵니다. 

Driver는 팀에 소속되어 있을 테니,  JSON DV에 team 데이터를 추가해 보겠습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2025.png)

테이블을 선택하면, parent / child 관계의 테이블을 추가할 수 있습니다. Add paraent Table을 선택합니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2026.png)

테이블의 FK관계를 보고, tool에서 Team 테이블을 목록에 띄워줍니다. 선택하여 반영합니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2027.png)

driver와 team 데이터를 모두 가지는 JSON DV 생성스크립트가 작성되었습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2028.png)

하단의 Test Query 를 선택하면 데이터도 볼수 있습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2029.png)

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2030.png)

추가를 원하는 컬럼이 있다면, 테이블 다이어그램에서 컬럼의 박스에 체크하면 스크립트에 반영됩니다.

이대로 생성하고 싶다면, 오른쪽의 파란버튼 - Create Duality View를 선택합니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2031.png)

TEST_DV라는 새로운 JSON DV 를 GUI기반으로 간단하게 만들어 보았습니다. 

![image.png](%5BDB%5D%20Oracle%20DB%2023ai%20JSON%20Relational%20Duality%20HoL%20v2/image%2032.png)

## 맺음말

이렇게 Oracle Realtional Duality View의 주요 기능을 실습을 통해 알아보았습니다.

오라클 DB 23ai 와 ORDS 를 통해 기존 오라클 개발자, 그리고 NoSQL 이나 Document DB에 익숙한 개발자들은 

- 데이터의 효율적인 저장
- 손쉬운 어플리케이션의 데이터 접근
- 데이터 일관성 문제 해결
- 고도화된 SQL을 활용한 분석

을 오라클 DB 에코시스템 안에서 손쉽게 구현할 수 있습니다. 

<br><br>
