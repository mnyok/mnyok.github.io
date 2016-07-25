---
layout: post
title:  "[Sqlite]pagerRollbackWal()"
date:   2016-07-25 22:00:00 +0000
categories: sqlite
tags: sqlite
---

- `pagerPlaybackSavepoint`에서 `pSavepoint == 0`이고 wal모드일 경우 호출된다.

- 캐시 안의 페이지 중 dirty이거나, 커밋되지 않은 채로 로그에 쓰여진 모든 페이지에 대해 두 가지 행동 중 하나를 한다.

  - `refcount==0`인 경우: 캐시된 페이지를 삭제한다.
  - `refcount>0`인 경우: 페이지 내용을 데이터베이스에서 다시 가져온다.

1. `sqlite3WalUndo`를 호출한다. 정확한 내부 기능과 pagerUndoCallback의 용도는 살펴본 후 수정

2. `sqlite3PcacheDirtyList`를 통해 캐시 안의 모든 dirty 페이지를 페이지 넘버로 정렬해 pList에 저장한다.

3. pList를 따라가면서 리스트 노드의 페이지 넘버에 대해 pagerUndoCallBack을 호출한다.
