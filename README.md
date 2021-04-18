# Python-stock-investment-automation
유튜브 조코딩님의 파이썬 주식 투자 자동화

전략

- 변동성 돌파 전략 - Larry R. Williams

	```python
	def get_target_price(code) # 목표가를 구하는 함수
	```
	

- 이동평균선 5일 + 10일

  ```python
  def get_movingaverage(code, window)
  ```

ETF(상장지수 펀드) 자동매매(주식 보다 판매 수수료가 저렴)

LP(유동성 공급자) 활동 기간: 0905 ~ 1520

자동매매 시간: 0905 ~ 1515

주문 호가

- 최유리
  - 당장 가장 유리하게 매매할 수 있는 가격
- 최호가
  - 우선 대기하는 가격

주문 조건

- IOC
  - 체결 후 남은 수량 취소
- FOK
  - 전량 체결되지 않으면 주문 자체를 취소

최유리 + FOK

자동스케줄러 사용(프로그램 자동시작, 종료)

- 새 작업 만들기



## 프로젝트 환경 설정

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



## Slack Bot 만들기
### Slack 환경 설정
#### 1. 워크스페이스 만들기
- slack 홈페이지(https://slack.com/intl/ko-kr/) 좌측 상단의 **SLACK 실행** 클릭 → 새 워크스페이스 생성

#### 2. slack-bot

- slack api 접속(https://api.slack.com/) → **Create a custom app** 클릭
- Slack App의 이름과 워크스페이스 지정 후 **Create App** 클릭
- **Your Apps**에 내가 만든 봇의 **Basic Infomation** 좌측의 **OAuth & Permissions** 클릭 → **Bot Token Scopes**에서 권한을 설정
  - **Add an OAuth Scope** 클릭 → **chat:write** 권한 설정 → 상단의 **Install to Workspace** 클릭(허용)
- 워크스페이스에서 만든 채널 - 세부정보(느낌표) - 더보기 - 앱추가 → 만든 봇을 추가



### 파이썬 Slacker 라이브러리

#### 1. Slacker 라이브러리

- slacker git 접속(https://github.com/os/slacker)

- 터미널을 사용한 설치: ``pip install slacker``, ``pip install requests``

- requests를 활용한 예제

  ```python
  import requests
   
  def post_message(token, channel, text):
      response = requests.post("https://slack.com/api/chat.postMessage",
          headers={"Authorization": "Bearer "+token},
          data={"channel": channel,"text": text}
      )
      print(response)
   
  myToken = "<your-slack-api-token-goes-here>"
  
  # Send a message to your channel 
  post_message(myToken,"<your-slack-chat-channel>", "Hello! World!")
  ```

  출처: https://developerdk.tistory.com/96



## 관련 소스코드

### CREON Plus API

#### 1. 주식정보 가져오기

- 소스코드 (출처: https://money2.creontrade.com/e5/mboard/ptype_basic/plusPDS/DW_Basic_Read.aspx?boardseq=299&seq=41&page=3&searchString=&prd=&lang=7&p=8833&v=8639&m=9505)

  ```python
  import win32com.client
   
  # 연결 여부 체크
  objCpCybos = win32com.client.Dispatch("CpUtil.CpCybos")
  bConnect = objCpCybos.IsConnect
  if (bConnect == 0):
      print("PLUS가 정상적으로 연결되지 않음. ")
      exit()
   
  # 현재가 객체 구하기
  objStockMst = win32com.client.Dispatch("DsCbo1.StockMst")
  objStockMst.SetInputValue(0, 'A005930')   #종목 코드 - 삼성전자
  objStockMst.BlockRequest()
   
  # 현재가 통신 및 통신 에러 처리 
  rqStatus = objStockMst.GetDibStatus()
  rqRet = objStockMst.GetDibMsg1()
  print("통신상태", rqStatus, rqRet)
  if rqStatus != 0:
      exit()
   
  # 현재가 정보 조회
  code = objStockMst.GetHeaderValue(0)  #종목코드
  name= objStockMst.GetHeaderValue(1)  # 종목명
  time= objStockMst.GetHeaderValue(4)  # 시간
  cprice= objStockMst.GetHeaderValue(11) # 종가
  diff= objStockMst.GetHeaderValue(12)  # 대비
  open= objStockMst.GetHeaderValue(13)  # 시가
  high= objStockMst.GetHeaderValue(14)  # 고가
  low= objStockMst.GetHeaderValue(15)   # 저가
  offer = objStockMst.GetHeaderValue(16)  #매도호가
  bid = objStockMst.GetHeaderValue(17)   #매수호가
  vol= objStockMst.GetHeaderValue(18)   #거래량
  vol_value= objStockMst.GetHeaderValue(19)  #거래대금
   
  # 예상 체결관련 정보
  exFlag = objStockMst.GetHeaderValue(58) #예상체결가 구분 플래그
  exPrice = objStockMst.GetHeaderValue(55) #예상체결가
  exDiff = objStockMst.GetHeaderValue(56) #예상체결가 전일대비
  exVol = objStockMst.GetHeaderValue(57) #예상체결수량
   
   
  print("코드", code)
  print("이름", name)
  print("시간", time)
  print("종가", cprice)
  print("대비", diff)
  print("시가", open)
  print("고가", high)
  print("저가", low)
  print("매도호가", offer)
  print("매수호가", bid)
  print("거래량", vol)
  print("거래대금", vol_value)
   
   
  if (exFlag == ord('0')):
      print("장 구분값: 동시호가와 장중 이외의 시간")
  elif (exFlag == ord('1')) :
      print("장 구분값: 동시호가 시간")
  elif (exFlag == ord('2')):
      print("장 구분값: 장중 또는 장종료")
   
  print("예상체결가 대비 수량")
  print("예상체결가", exPrice)
  print("예상체결가 대비", exDiff)
  print("예상체결수량", exVol)
  ```




### TRADE

#### 1. GitHub 

- github: https://github.com/INVESTAR/StockAnalysisInPython/blob/master/08_Volatility_Breakout/ch08_03_EtfAlgoTrader.py



### CREON Plus 자동 로그인

#### 1. GitHub

- github: https://github.com/INVESTAR/StockAnalysisInPython/blob/master/08_Volatility_Breakout/ch08_01_AutoConnect.py

