---
layout: post
title:  "[Sqlite]pagerUndoCallback()"
date:   2016-07-25 22:21:00 +0000
categories: sqlite
tags: sqlite
---

## 기능 개요

dirty 페이지를 버리거나(discard) DB로 복구(reload)한다.

## 상세

- `assert( pagerUseWal(pPager) );`가 존재한다. 즉 이 함수는 wal모드에서만 실행된다.

- `pCtx`에는 `pPager`가, `iPg`에는 해당 페이지의 페이지 넘버가 전달된다.

- `pagerRollbackWal`에서 dirty 페이지들에 대해 각각 한번씩 호출된다.

- 페이지가 cache 안에 존재하지 않을 경우, 아무 일도 하지 않고 `sqlite3BackupRestart`만 실행한 채 함수를 종료한다. 해당 페이지는 버려진다.

- 하나 이상의 outstanding reference가 존재할 경우, 페이지는 DB로 복구(reload)된다.


- `sqlite3BackupRestart`: non-WAL의 경우 롤백 저널의 데이터가 DB로 복사되지만, WAL에서는 로그 파일을 자르는 것으로 롤백이 실행된다. 만약 어떤 프레임이 현재 트랜젝션에 의해 로그에 쓰여서 DB로 복사됐다면, backup이 반드시 다시 시작되어야 한다...고 쓰여 있다. 우리의 롤백 방식에도 적용되어야 하는지 생각이 필요하다.
