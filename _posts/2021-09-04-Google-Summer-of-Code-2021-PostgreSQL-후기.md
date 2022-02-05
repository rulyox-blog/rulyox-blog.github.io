---
layout: post
title: Google Summer of Code 2021 PostgreSQL 후기
thumbnail-img: /assets/img/posts/2021-09-04-Google-Summer-of-Code-2021-PostgreSQL-후기/thumb.png
share-img: /assets/img/posts/2021-09-04-Google-Summer-of-Code-2021-PostgreSQL-후기/thumb.png
tags: [Talks]
comments: true
---

2021년 5월부터 8월까지 [Google Summer of Code](https://summerofcode.withgoogle.com/) 프로그램에 참여하였습니다.

Google Summer of Code는 Google에서 주최하고, 대학생들이 오픈소스 기관의 프로젝트에 참여하여 여름동안 프로그래밍을 하는 프로그램입니다.

![certificate.png](/assets/img/posts/2021-09-04-Google-Summer-of-Code-2021-PostgreSQL-후기/accepted.png){: .mx-auto.d-block :}
<p align="center"><i>Accepted</i></p>

## PostgreSQL Performance Farm

전 PostgreSQL의 [Performance Farm](https://summerofcode.withgoogle.com/archive/2021/projects/5285139025756160) 프로젝트에 참여하였습니다.

Performance Farm은 PostgreSQL 소스 코드를 다운로드하여 빌드하고, 벤치마크를 수행하여 결과를 수집한 뒤, 결과를 탐색힐 수 있는 웹사이트를 제공하는 과정을 자동화하는 프로젝트입니다.

3명의 멘토였던 Ilaria Battiston, Stephen Frost, Andreas Scherbaum께서 방향성을 잘 잡아주시고 다양한 의견을 주셔서 프로젝트를 성공적으로 마무리 지을 수 있었습니다.

이전에는 큰 오픈소스 프로젝트에 기여한 경험이 없었는데, GSoC을 통해 오픈소스의 세계에 한 발을 내디딘 것 같아 뿌듯했습니다.

![certificate.png](/assets/img/posts/2021-09-04-Google-Summer-of-Code-2021-PostgreSQL-후기/certificate.png){: .mx-auto.d-block :}
<p align="center"><i>Certificate</i></p>

## Proposal 작성 팁

전 인터넷으로 여러 GSoC proposal 작성 팁을 검색해봤고, 그 결과 첫 도전에 합격할 수 있었습니다.

아래는 제가 느끼기에 합격에 큰 영향을 미치는 팁들입니다.

- GSoC proposal 작성 전 미리 해당 오픈소스 프로젝트에 기여 (하지만 이것은 난이도가 있는 일이고, 어떤 프로젝트가 있을지도 모르므로 현실적으로 어렵습니다. 저도 미리 기여하지는 않았습니다.)
- 오픈소스 기관 채팅방(주로 Slack)에 들어가 커뮤니티에 참여하고, 해당 프로젝트의 예정된 멘토가 누구인지, 다른 지원자들이 어떤지 파악
- Student 선발은 Google과는 무관하고 멘토들이 선발하기 때문에, 멘토들이 원하는게 무엇인지 파악
- 일반적으로 멘토가 작성한 프로젝트에 대한 설명이 충분하지 않은 경우가 많기 때문에, 명확하지 않은 부분이 있는 경우 커뮤니티에 질문 (질문함으로써 이 프로젝트에 관심이 있다는 것과 프로젝트에 대해 파악하기 위해 노력 중이라는 것을 표현할 수 있습니다.)
- Proposal을 빠르게 작성하고 멘토(멘토가 확정이 되지 않은 경우, 기관 어드민)에게 피드백 요청
- Proposal에 1~2주 단위로 쪼개진 일정을 첨부하여 어떤 식으로 진행할 것인지 작성
