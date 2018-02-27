---
layout: "post"
title: "Automatic ransomware testing in VM"
date: "2018-02-27 18:35"
category: Forensic
---

2년전부터 해온 연구가 있다.  
5명으로 시작해서 현재 나 포함 3명과 함께 진행하는 이 연구는  
랜섬웨어의 행위를 파악하여 사후 탐지, 나아가 IDS 개념의 조기 차단 시스템 구현이 목표였다.  

랜섬웨어의 행위는 파일을 암호화하고 돈을 요구하는게 기본적이지만(근데 아닌 애들도 있다. 별별 또라이같은 랜섬도 있음)low 레벨에서 남는 흔적들은 다양하기 때문에 직접 다 돌려보면서 눈으로 확인할 수 밖에 없었다. 
 
그런데 문제는 지금껏 샘플을 돌릴 때 다음처럼 모두 수동으로 돌렸었다.

0. 네트워크가 차단된 환경에서 따로 구한 랜섬웨어 샘플을 USB를 통해 준비하고
1. 관리자 권한으로 실행시킨다. 그럼 또 도는 애들이 있고 안 도는 애들도 있고 잠복하는 애들도 있고 돌듯 말듯 애매한 애들도 있다. 그래서 시간을 두고 기다려야 한다.
  - 그리고 이때 메모장에다가 실행 시간과 랜섬 이름을 기록한다. (랜섬웨어에 의해 기록또한 암호화 될 수 있으므로 3번단계 까지 프로세스는 끄지 않는다.)
2. 충분한 시간이 흐르면 리소스 모니터와 Sentinel file(랜섬웨어가 돌았는지 확인하기 위한 파일, 기본 OS에 존재하는 koala.jpg를 주로 보았다. Dummy or Decoy file은 아님에 유의)을 통해 이 랜섬웨어가 성공적으로 돌았는지 아니면 어딘가 잘못되었는지를 어느정도 파악할 수 있다.
3. 성공적으로 암호화가 진행되었다 판단되면 PowerShell 모듈인 PowerForensicsv2를 통해 UsnJrnl을 파싱한다.
4. 그리고 로그 값이랑 기록 들을 따로 USB로 빼온다.
5. 고스트를 이용해 환경을 복구한다.


보면 알겠지만 단순 반복 노동임에도 불구하고 상당히 사람 손이 많이 가는걸 알 수 있다. 음.. 사실 손이 많이 간다기보다 시간을 많이 잡아먹는다.


그럼에도 불구하고 지금껏 이런식으로 해온 이유는 그냥 조금 삽질하는게 기반을 다지기위해 삽질하는것보다는 임시적이지만 더 편했기 때문이었다. 그러나 얼마전부터 상황이 바뀌었다. 이 방식대로 계속 하다간 새로 들어오는 샘플을 감당할 수 없게 된다. 따라서 실험 설계의 기반을 싹 바꾸기로 했다.
