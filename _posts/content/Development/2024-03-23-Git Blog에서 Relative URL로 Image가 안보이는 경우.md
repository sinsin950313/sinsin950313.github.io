---
title: "Git Blog에서 Relative URL로 Image가 안보이는 경우"
date: 2024-03-23
categories: ["Development", "Git Blog"]
tags: ["Git Blog", "Relative URL Image", "protocol-relative URL"]
---

```
image link //assets/img/Title.jpg is a protocol-relative URL, use explicit https:// instead
```
갑자기 Git Blog에서 protocol-relative URL이라면서 Relative한 위치로 등록된 Image가 포함된 Posting이 안올라가는 현상이 발생.

당장 2주전만 해도 문제없이 동일한 방식으로 Posting을 했기 때문에 내가 사용하는 Chirpy든, Jekyll이든 Git Blog든 어딘가에 뭔가 수정이 발생한 듯.

현재 가장 간단한 해결책은

```
https://raw.githubusercontent.com/User/User.github.io/main
```

_config.yml에 img_cdn으로 Github에서 사용하는 CDN을 추가하는 방법

위 링크에서 User부분만 자신에 맞게 수정하면 됨.

정확하게는 Github의 Image가 저장된 URL에서 이미지를 우클릭해서 이미지를 새탭으로 열 경우 위 링크에 해당 이미지 파일 이름과 확장자가 적힌 URL로 연결되므로 파일 이름과 확장자를 제외한 부분을 복사한 후 img_cdn에 추가하면 일단 급한 문제는 해결됨.

단, Github의 Image는 확장자를 자동으로 대문자로 변환하는 듯 한데 만약 기존에 적힌 이미지 파일의 확장자가 소문자일 경우는 검색되지 않음.

직접 수정하든가.. 다른 방법을 찾던가.. 다시 원래대로 돌아가기를 기다리던가..
