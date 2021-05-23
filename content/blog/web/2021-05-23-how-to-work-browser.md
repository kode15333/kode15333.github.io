---
title: "Modern Web Browser 톹아보기"  
description: "Modern Web Browser(Chrome) 톹아보기"
draft: false 
template: "post"
category : "web"
tags:
- browser
date: 2021-05-22

---
### 0. 글을 작성하는 이유

- 웹 브라우저의 동작 원리에 대해서 알고 싶어서
- 웹 브라우저의 여러가지 쓰레드에 대해서 알고싶어서
- 너무나도 좋은 자료인데, 정리를 해서 나의 것으로 만들고 싶어서

### 1. 사전 지식 : Process, Thread, IPC

![https://developers.google.com/web/updates/images/inside-browser/part1/process-thread.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part1/process-thread.png?hl=ko)

![https://developers.google.com/web/updates/images/inside-browser/part1/workerprocess.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part1/workerprocess.png?hl=ko)

- Process(프로세스)
    - 보조기억장치(HDD)에 위치한 프로그램이 메모리에 올라온 상태
    - OS는 프로세스에게 메모리를 할당하여 해당 프로세스의 정보를 고유한 메모리 공간(PCB)에 저장할 수 있게 한다.
    - 자원소유의 단위(메모리가 할당되었다)
- Thread(쓰레드)
    - 프로세스 내부에 있으며 프로세스를 실행하는 주제
    - Dispatching의 단위(CPU에게 계산/연산 처리를 요청하는 단위)
- IPC(Inter Process Comunication)
    - 프로세스는 다른 프로세스를 돌려서 별도의 작업을 수행하도록 OS에 요청 할 수 있다.

      (OS는 새로운 프로세스에게 별도의 메모릴 공간을 할당한다)

    - 두 프로세스간 통신이 필요하다면, IPC를 이용한다(데이터 교환)

### 2. 웹 브라우저에는 어떤 프로세스가 있나?

![https://developers.google.com/web/updates/images/inside-browser/part1/browserui.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part1/browserui.png?hl=ko)

- 메인 프로세스와 여러개의 작은 프로세스가 통신하면서(IPC) 작동
- 프로세스의 정의와 역할
    1. 브라우저(Main) - 브라우저의 전체적인 제어 및 탭 밖에 있는 모든 프로세스를 담당
        - 브라우저의 버튼이나 입력창을 그리는 **UI 쓰레드**
        - 데이터를 수신하는 **네트워크 쓰레드**
        - 파일 같은것들을 접근하기 위한 **스토리지 쓰레드**
    2. 렌더러 - 웹 사이트가 디스플레이 될때 사용되며, 각 탭 별로 프로세스가 존재
        - 각 탭로 별도의 프로세스로 관리되기 때문에 하나가 멈춰도 다른 탭은 정상 작동
    3. 플러그인 - 리엑트 개발자 도구 같은 플러그인 관리
    4. GPU - 다른 프로세스와 분리된 GPU 작업을 제어
        - 동일한 표면(창)에 표시하기 때문에 별도로 분리

### 3. jooonho.com을 치면 어떻게 작동이 될까?

1. 입력 처리
    - 사용자가 주소창에 입력하기 시작하면 UI 쓰레드가 검색어인가? URL인가? 판단

      ![https://developers.google.com/web/updates/images/inside-browser/part2/input.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/input.png?hl=ko)

2. 탐색 시작
    - 엔터를 치면, UI 쓰레드가 컨텐츠를 내려받기 위해 네트워크 쓰레드에게 요청
    - 네트워크 쓰레드에게 요청할때, UI 쓰레드가 어떤 렌더러 프로세스가 받을지 준비한다.

      ![https://developers.google.com/web/updates/images/inside-browser/part2/navstart.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/navstart.png?hl=ko)

3. 응답 읽기
    - 네트워크 쓰레드가 응답 파일을 체크해서 해당하는 프로세스에게 데이터 이관
        - HTML ⇒ 해당 탭의 렌더러 프로세스에게 이관
        - zip 또는 다른 형식의 파일 ⇒ 다운로드 매니저에게 이관
    - 넘기기전, CORB 체크(악성 사이트 여부)로 경고창 노출

      ![https://developers.google.com/web/updates/images/inside-browser/part2/sniff.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/sniff.png?hl=ko)

4. 렌더러 프로세스 찾기
    - 네트워크 쓰레드를 통해 UI 쓰레드에게 데이터를 준비되었다고 통보
    - UI쓰레드는 해당 페이지를 그릴 렌더러 프로세스를 찾는다

![https://developers.google.com/web/updates/images/inside-browser/part2/findrenderer.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/findrenderer.png?hl=ko)

5. 탐색 수행

- 브라우저 프로세스는 IPC를 통해 렌더러 프로세스에게 해당 데이터를 전송
- 필요하다면, 데이터 스트림을 전달하여, 렌더러 프로세스가 HTML 데이터를 계속 받을 수 있게 한다.
- 렌더러 프로세스에서 커밋이 확인되면, 브라우저 프로세스는 탐색을 완료하고 문서 로딩 단계를 시작한다

  ![https://developers.google.com/web/updates/images/inside-browser/part2/commit.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/commit.png?hl=ko)

6. 초기 로딩 완료

- 탐색이 커밋되고 나며, 렌더러 프로세스가 리소스 로딩과 페이지 렌더를 지속
- 렌더러 프로세스가 렌더링을 끝내면, 브라우저 프로세스에게 IPC를 반환
- UI쓰레드가 탭의 로딩 스피너를 정지

  ![https://developers.google.com/web/updates/images/inside-browser/part2/loaded.png?hl=ko](https://developers.google.com/web/updates/images/inside-browser/part2/loaded.png?hl=ko)
