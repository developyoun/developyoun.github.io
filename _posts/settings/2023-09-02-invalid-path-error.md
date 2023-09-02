---
title: invalid path xxx Error

categories:
  - settings 

tags:
  - settings

toc: true
toc_sticky: true
toc_label: 'Contents'
---

## 원인
이는 **윈도우** 환경에서 발생할 수 있는데, Clone을 시도하는 **파일의 이름에 특수문자**가 들어가서 발생한 것이다.
> 저의 케이스인 경우에 Mac에서 특수문자가 들어간 파일을 커밋해서 window로 pull을 받다가 발생했습니다.

## 조치
`git config` 설정을 통해 이를 해결할 수 있다.
git bash 를 켜서 아래의 명령을 입력하면 된다.
```bash
git config core.protectNTFS false
git checkout -f HEAD
```

이후에는, 특수문자가 포함된 파일명을 제외하고 pull을 받는다.
> 특문이 있는 파일은 해결되지 못하는 듯;

> 레퍼런스
> https://jkim83.tistory.com/148

