# DB

## SQLコマンド関連
### ${dbuserp}QL
接続
```
psql --host fr-${dbuserp}ql-db.${dbstring}.ap-northeast-1.rds.amazonaws.com --port=5432 --username=${dbuser} --password --dbname=${dbname}
```

ロールの確認
```
\du
```
	
ロールの追加
```
CREATE ROLE ロール名 LOGIN PASSWORD 'XXX';
CREATE ROLE ${dbuser} LOGIN PASSWORD ${dbpass};
```

ロールにログイン権限付与
```
alter role ${dbuser} login;
```

ロールにパスワード付与
```
alter role <RoleName> password 'XXXXX';
alter role ${dbuser} password ${dbpass};
```

ロールにDB作成権限付与
```
alter role ${dbuser} createdb;
```

ロールの削除
```
drop role ロール名;
```

DB表示
```
\l
```

DB作成
```
create database ${dbname} with owner ${dbuserp} encoding 'UTF8' lc_collate 'ja_JP.UTF-8' lc_ctype 'ja_JP.UTF-8' template=template0;
```

オーナ変更
```
#alter database <DBName> owner to <roleName>;
alter database ${dbname} owner to ${dbuser};
```

使用DBの変更
```
\c <DBName>
\c ${dbname}
```

テーブル一覧、権限付与状況の確認
```
\z
```

ユーザ一覧
```
\du
```

接続中のDB情報
```
\conninfo
```

Update
```
update user_profile sete email = 'xxx@xxx' whre uid = '280025';
update location_status set location_status = 'out' where lsid = '264';
```
Insert
```
insert into group_locations (id, state, location_id, group_id, created_at, updated_at) values (20,1,4,10,current_timestamp,current_timestamp);

insert into temperatures (updated_at) values (('2021-06-27T01:02:10Z') at time zone 'JST');
```

テーブルデータをバックアップ
```
select * into hr_members_backup from hr_members;
```

テーブル削除
DROP TABLE IF EXISTS <tableName>

シーケンス削除
DROP SEQUENCE IF EXISTS <sequenceName>


### SQL
シーケンス作成
```
CREATE SEQUENCE temperatures_tid_seq;
```

テーブル作成
```
CREATE TABLE temperatures (
    tid           integer PRIMARY KEY DEFAULT nextval('temperatures_tid_seq') not null,
    degree        real,
    time          timestamp,
    created_at    timestamp,
    created_by    text,
    updated_at    timestamp DEFAULT current_timestamp,
    updated_by    text
);
GRANT ALL ON locations, temperatures_tid_seq TO ${dbuser};

```

```
CREATE SEQUENCE humiditys_hid_seq;

CREATE TABLE humiditys (
    hid           integer PRIMARY KEY DEFAULT nextval('humiditys_hid_seq'),
    degree        real,
    time          timestamp,
    created_at    timestamp,
    created_by    text,
    updated_at    timestamp DEFAULT current_timestamp,
    updated_by    text
);
```

```
CREATE SEQUENCE illuminances_iid_seq;

CREATE TABLE illuminances (
    iid           integer PRIMARY KEY DEFAULT nextval('illuminances_iid_seq'),
    degree        real,
    time          timestamp,
    created_at    timestamp,
    created_by    text,
    updated_at    timestamp DEFAULT current_timestamp,
    updated_by    text
);
GRANT ALL ON locations, illuminances_iid_seq TO ${dbuser};
```

```
CREATE SEQUENCE locations_lid_seq;

CREATE TABLE locations (
    lid                  integer PRIMARY KEY DEFAULT nextval('locations_lid_seq'),
    time                 timestamp,
    latitude             decimal,
    longitude            decimal,
    horizontalaccuracy   decimal,
    floorlevel           decimal,
    altitude             decimal,
    created_at           timestamp,
    created_by           text,
    updated_at           timestamp DEFAULT current_timestamp,
    updated_by           text
);

GRANT ALL ON locations, locations_lid_seq TO ${dbuser};
```

```
CREATE SEQUENCE weathers_wid_seq;

CREATE TABLE weathers (
    wid                  integer PRIMARY KEY DEFAULT nextval('weathers_wid_seq'),
    temp                 decimal,
    humidity             decimal,
    time                 timestamp,
    feels_like           decimal,
    pressure             decimal,
    dew_point            decimal,
    uvi                  decimal,
    wind_speed           decimal,
    wind_deg             decimal,
    wind_gust            decimal,
    weather              text,
    created_at           timestamp,
    created_by           text,
    updated_at           timestamp DEFAULT current_timestamp,
    updated_by           text
);

GRANT ALL ON weathers, weathers_wid_seq TO ${dbuser};
```
```
CREATE SEQUENCE location_status_lsid_seq;

CREATE TABLE location_status (
    lsid                 integer PRIMARY KEY DEFAULT nextval('location_status_lsid_seq'),
    lid                  integer REFERENCES locations(lid),
    location_status      text,
    location_at          text,
    created_at           timestamp,
    created_by           text,
    updated_at           timestamp DEFAULT current_timestamp,
    updated_by           text
);

GRANT ALL ON location_status, location_status_lsid_seq TO ${dbuser};
```