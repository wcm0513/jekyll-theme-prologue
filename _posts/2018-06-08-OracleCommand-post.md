---
title: ORACLE 명령어
author: wcm
layout: post
---

●오라클 명령어
(필요에 따라 계속 수정)

[계정 생성]
create user 계정명 identified by 비밀번호

[user에 권한 주기]
GRANT connect, resource, dba TO [user_name];

[현재 접속자]
show user

[db에 유저 확인]
select username from dba_users;

[현재 계정의 테이블 목록]
select * from tab;

[계정 잠김 확인]
SELECT username, account_status, lock_date FROM dba_users;

[계정 잠금 해제]
ALTER USER scott ACCOUNT UNLOCK;

[테이블스페이스 조회]
SELECT TABLESPACE_NAME, STATUS, CONTENTS FROM DBA_TABLESPACES;

[테이블스페이스 생성]
create tablespace 테이블명
datafile 'C:\oraclexe\app\oracle\oradata\XE/파일명.dbf' size 500m;

[테이블스페이스 삭제]
drop tablespace 테이블명
including contents and datafiles
cascade constraints;

[파티셔닝 사용 여부 확인]
select * from v$option where parameter = 'Partitioning';
-personal과 enterprise 버젼에서 지원함

[버전 정보 확인]
select * from v$version;

[오라클 데이터베이스명 확인]
SELECT NAME, DB_UNIQUE_NAME FROM v$database;

[오라클 SID 확인]
SELECT instance FROM v$thread;