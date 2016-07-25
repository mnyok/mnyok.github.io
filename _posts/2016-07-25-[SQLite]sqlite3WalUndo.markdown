---
layout: post
title:  "[Sqlite]sqlite3WalUndo()"
date:   2016-07-25 22:34:00 +0000
categories: sqlite
tags: sqlite
---

### 기능 개요

1. 로그 파일에 쓰인 채로 커밋되지 않은 데이터가 남아있을 경우, write-pointer를 트랜잭션의 시작으로 옮긴다.

2. 추가로 트랜잭션의 시작부터 쓰인 각 프레임에 대해 callback function이 실행된다.

### 내부

```c
memcpy(&pWal->hdr, (void \*)walIndexHdr(pWal), sizeof(WalIndexHdr));

for(iFrame=pWal->hdr.mxFrame+1;
    ALWAYS(rc==SQLITE_OK) && iFrame<=iMax;
    iFrame++
){
  assert( walFramePgno(pWal, iFrame)!=1 );
  rc = xUndo(pUndoCtx, walFramePgno(pWal, iFrame));
}
if( iMax!=pWal->hdr.mxFrame ) walCleanupHash(pWal);

```

1. `memcpy`로 `pWal->hdr` 안에 트랜젝션이 일어나기 전의 헤더를 복구한다.

2. `hdr.mxFrame+1`부터 `iFrame<=iMax`까지 `xUndo`가 실행된다.
>hdr.mxFrame의 값을 변경하는 것으로 wal파일 헤더에 마스터저널 정보를 추가할 수 있는가?

  - 프로젝트를 뒤져봐도 `xUndo`의 선언부가 보이지 않는다. 검색해도 별달리 나오는 것이 없다.

  - 주석에서 `xUndo`는 절대 실패(fail)하지 않는다고 한다. rc는 항상 `SQLITE_OK`를 리턴받는 듯.

3. 모든 프레임을 지난 뒤 mxFrame값이 변경됐을 경우 `walCleanupHash`를 호출한다.
