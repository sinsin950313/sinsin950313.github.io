---
title: "Velog to Git Blog"
date: 2024-01-17
categories: ["Development", "portfolio"]
tags: ["cpp", "portfolio"]
---

# Velog Posting을 Github Blog로 Porting하기
Velog Crawling해서 다운로드 하는 프로그램은 이걸 사용했습니다.
<br>
[Link](https://github.com/cjaewon/velog-backup)

다운로드한 포스트들을 사용하는 Jekyll Theme에 맞도록 약간 수정을 하는 프로그램입니다.

제가 사용하는 Jekyll Theme 입니다.
<br>
[Link](https://chirpy.cotes.page/)

## 기술 스택
- CPP
- File IO

## 사용법
cmd에서 Exe 파일 위치를 기준으로

```
a.out -c {CategoryFile} -s {SourceDirectory} -d {DestinationDirectory}
```

도움말
```
a.out -h
```

## CategoryFile
```
[Category, {Tag1, Tag2, ..., TagN}]
```
제가 사용하는 Jekyll Theme의 경우 Super Category와 Sub Category를 요구하는데 Super Category는 Development로 고정입니다.

CategoryFile에 작성되지 않은 Tag들은 Sub Category로 Non이 사용됩니다.

[Source](https://github.com/sinsin950313/VelogToGitBlog)