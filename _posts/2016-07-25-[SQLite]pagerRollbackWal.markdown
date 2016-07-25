---
layout: post
title:  "[Sqlite]pagerRollbackWal()"
date:   2016-07-25 22:00:00 +0000
categories: sqlite
tags: sqlite
---

## 기능 개요

WAL DB의 트랜젝션을 롤백하기 위해 호출된다

## 상세

- `pagerPlaybackSavepoint`에서 `pSavepoint == 0`이고 wal모드일 경우 호출된다.

- 캐시 안의 페이지 중 dirty이거나, 커밋되지 않은 채로 로그에 쓰여진 모든 페이지에 대해 두 가지 행동 중 하나를 한다.

  - `refcount == 0`: 캐시된 페이지를 삭제한다.
  - `refcount > 0`: 페이지 내용을 데이터베이스에서 다시 가져온다.

1. `sqlite3WalUndo`를 호출한다. 페이지에 대한 실질적인 작업이 일어나는 듯. [해당 글](http://orc1226.github.io/2016/07/SQLite-sqlite3WalUndo) 참조.

2. `sqlite3PcacheDirtyList`를 통해 캐시 안의 모든 dirty 페이지를 페이지 넘버로 정렬해 pList에 저장한다.

3. pList를 따라가면서 리스트 노드의 페이지 넘버에 대해 pagerUndoCallBack을 호출한다.
