# Python-stock-investment-automation
유튜브 조코딩님의 파이썬 주식 투자 자동화



## 환경설정

### 1. 크레온 HTS 설치 (+계좌개설)

#### 1. 비대면계좌개설

- 크레온 App (Mobile) → 비대면계좌개설

#### 2. 크레온 API 사용 설정 (윈도우 환경 필수)

- 대신증권 크레온 로그인(https://www.creontrade.com/) → 온라인지점 → 서비스신청관리 → 시스템트레이딩
- 신청 → 시스템트레이딩신청

#### 3. 크레온 HTS 설치

- 대신증권 크레온 로그인(https://www.creontrade.com/) → 고객라운지 →  → 트레이딩 안내 → 다운로드 센터 → CREON HTS 다운로드 → 설치 
- 크레온 실행 → creon plus 로그인
- CreonPlus 실행 → 트레이로 실행된 CreonPlus Start 우클릭 → 주문 오브젝트 사용 동의 → 주문 내역 확인 설정 → 주문내역 확인 **체크 해제**    

### 2. 파이썬 설치 & 라이브러리 세팅

#### 1. 파이썬 3.8 설치 (32bit, 증권사 API를 사용하기 위함)

- https://www.python.org/ → Downloads → Windows → Download Windows x86 excutable installer
- Install Python 3.8.6 (32bit) → 하단의 체크 박스 Add Python 3.8 to PATH 체크 → Customize installation → Option Features 모두 체크 → Advance Option 모두 체크, 설치 경로(C:\python38-32) → Install
- 시스템 환경변수 편집 → 환경 변수 → 시스템 변수 - Path 편집 → python38-32 경로를 맨 위로 이동
- cmd에서 ``python`` 을 입력하면 설치한 버전의 파이썬 사용가능
- 설치한 파이썬을 관리자 권한으로 설정: 설치한 경로(C:\python38-32)의 python.exe와 pythonw.exe를 우클릭 → 호완성 탭의 **관리자 권한으로 이 프로그램 실행** 체크



#### 2. Visual Code 세팅

- 코드를 작성할 **Visual Code** 설치 및 관리자 권한으로 실행

- Visual Code의 Teminal 탭 클릭 → Select Default Profile → command Prompt 클릭 
  (터미널이 cmd로 안보일 경우 터미널을 닫고 다시 열기)

- API 사용에 필요한 라이브러리 설치

  - 윈도우 작업 자동화 라이브러리: ``pip install pywinauto``
  - Visual Code 우측 하단에 권장하는 Python Extensions을 설치, 설치한 버전에 맞는 Python Interpreter 선택(좌측 하단 Python 버전)

- test.py 파일을 만들어서 예제 테스트

- 크레온플러스의 자료실에서 **[파이썬] 종목정보 구하는 예제** 코드 사용
  (출처: https://www.creontrade.com/g.ds?m=9505&p=8815&v=8633)

  ```python
  import win32com.client 
   
  # 연결 여부 체크
  objCpCybos = win32com.client.Dispatch("CpUtil.CpCybos")
  bConnect = objCpCybos.IsConnect
  if (bConnect == 0):
      print("PLUS가 정상적으로 연결되지 않음. ")
      exit()
   
  # 종목코드 리스트 구하기
  objCpCodeMgr = win32com.client.Dispatch("CpUtil.CpCodeMgr")
  codeList = objCpCodeMgr.GetStockListByMarket(1) #거래소
  codeList2 = objCpCodeMgr.GetStockListByMarket(2) #코스닥
   
   
  print("거래소 종목코드", len(codeList))
  for i, code in enumerate(codeList):
      secondCode = objCpCodeMgr.GetStockSectionKind(code)
      name = objCpCodeMgr.CodeToName(code)
      stdPrice = objCpCodeMgr.GetStockStdPrice(code)
      print(i, code, secondCode, stdPrice, name)
   
  print("코스닥 종목코드", len(codeList2))
  for i, code in enumerate(codeList2):
      secondCode = objCpCodeMgr.GetStockSectionKind(code)
      name = objCpCodeMgr.CodeToName(code)
      stdPrice = objCpCodeMgr.GetStockStdPrice(code)
      print(i, code, secondCode, stdPrice, name)
   
  print("거래소 + 코스닥 종목코드 ",len(codeList) + len(codeList2))
  ```

  실행 방법: 터미널 입력 ``python test.py``

