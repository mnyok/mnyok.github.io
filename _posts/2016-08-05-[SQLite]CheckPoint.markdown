---
layout: post
title:  "[Sqlite]CheckPoint"
date:   2016-08-05 09:11:00 +0000
categories: sqlite
tags: sqlite
---

WAL에서는 write가 일어날 때 직접 DB파일에 쓰지 않고 db-wal 파일에 데이터를 쓴다. 문제는 db-wal 파일이 너무 커질 경우 read시에 데이터를 찾느라 오랜 시간이 걸린다는 것이다. 리스트 형태의 wal 파일을 빠르게 탐색하기 위해 wal-index(db-shm 파일)을 사용하지만, 속도 저하를 완전히 막을수는 없다.

따라서 wal파일의 크기를 관리하기 위해 checkpoint를 사용해 wal 파일의 데이터를 db파일로 옮긴다. 기본값은 프레임이 1000개일때 checkpoint를 실행하고, 변경 가능하다. write를 하면서  wal의 크기가 1000개를 넘게 되면, 그 다음 커밋부터 checkpoint를 실행해 1000개 이하로 내려갈 때 까지 반복한다.

checkpoint는 write에서 일어나기 때문에 wal 파일 크기의 제한이 작을수록 write가 느려지고, read가 빨라진다. write의 속도 저하를 줄이기 위해서 commit시에 하는 대신 쓰레드나 프로세스를 따로 만들어 checkpoint를 담당하도록 할 수도 있다.

checkpoint는 4가지 모드로 실행된다.

- PASSIVE
  
  기본값. read와 write를 방해하지 않고, 방해가 되지 않는 프레임에 대해서만 checkpoint를 한다. 과정이 다 끝나도 wal 파일은 완전히 checkpoint되지 않은 상태로 남아있을 수 있다.

- FULL

  먼저 reader와 writer가 남지 않을때까지 막는다. 이후 모든 프레임에 대해 체크포인트를 진행하는데, 이때 새로 들어오는 reader는 허용한다.

- RESTART

  FULL과 동일한 방식으로 진행하지만, 새로 들어오는 reader가 DB파일만 읽을 수 있도록 제한한다. 체크포인트가 끝난 뒤 들어오는 writer는 wal의 처음부터 시작하게 된다.

- TRUNCATE

  RESTART와 같은 방식으로 진행하되, 과정이 끝나면 WAL파일을 0으로 초기화한다.

DB파일 뒤에 마스터저널 이름을 쓰는 방법의 문제는 기본값인 passive에서 발생했는데, 체크포인트가 write를 막지 않고 진행하기 때문에 실행되는 checkpoint의 시점을 예측할 수가 없다는 것이다. 마스터파일의 이름 뒤에 데이터가 추가되면 큰 문제가 생기므로, 다른 방법을 찾기로 했다.
