---
layout: post
title:  "[Sqlite]pagerPlaybackSavepoint"
date:   2016-07-12 21:01:47 +0000
categories: sqlite
tags: sqlite
---

>수정 진행중

#### pSavepoint가 NULL이 아닌 경우, pSavepoint를 Playback

non-transaction savepoint가 롤백된것을 의미한다. 롤백은 3단계로 이루어진다. wal이 아닌 경우에만 해당하는 설명으로 예상됨.  

1. ㅁ

2. ㅁ

3. 저널 파일을 통해서 실제 롤백이 일어난다. PagerSavepoint.iSubRec부터 저널 파일의 끝까지 이루어진다.  


#### pSavepoint가 NULL인 경우, 마스터 저널 파일 전체를 Playback

- ROLLBACK TO[^1] 커맨드가 transaction savepoint에 적용된 경우에 해당

- 페이지들은 메인 저널 파일만으로 playback된다고 한다.

- 이 경우엔 bitvec[^2]이 필요 없다.


```c
if( !pSavepoint && pagerUseWal(pPager) ){
    return pagerRollbackWal(pPager);
}
```

메소드 코드가 시작한지 얼마 되지 않아 wal을 빼내는 부분이 나온다. pSavepoint가 0일 경우 마스터 저널을 이용해서 롤백을 진행하기 때문에 wal의 경우 마스터 저널을 사용하지 않는 기존 방법을 쓰도록 하는 것 같다. 이 부분에서 우리가 만든 마스터저널을 사용할 수 있을 것 같다.  

- `pagerRollbackWal`은 `sqlite3WalUndo`를, `sqlite3WalUndo`에서는 `walCleanupHash`를 호출한다.  

이후 szJ에 메인 롤백 저널의 사이즈를 저장하는데, `assert( pagerUseWal(pPager)==0 || szJ==0 );` 부분으로 봐서 wal모드인 경우 szJ가 항상 0인 듯 하다. 예상이 맞다면 이후 페이지를 실제로 롤백하는 부분 `if( pSavepoint ){`로 넘어간다. 이곳에서 `sqlite3WalSavepointUndo`이 실행된다.  

그 뒤에는 `pager_playback_one_page` 함수가 있는데, wal모드에서도 실행되는지 확인 필요



[^1]: savepoint로 돌아가는 명령어. ROLLBACK 커맨드의 경우 transaction을 취소하고 돌아가지만, ROLLBACK TO는 transaction을 처음부터 다시 실행한다. 대신 그 사이의 savepoint들은 취소된다. <https://www.sqlite.org/lang_savepoint.html>  

[^2]: bitvec: bitvec.c에 정의되어 있다. 길이가 정해진 bitmap으로, bit는 1부터 숫자가 매겨진다. 트랜잭션 도중 어떤 페이지가 저널에 기록됐는지 체크한다.
