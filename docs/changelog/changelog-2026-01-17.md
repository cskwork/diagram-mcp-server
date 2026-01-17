# Changelog 2026-01-17

## Windows 호환성 수정 - 다이어그램 생성 타임아웃

### 문제
- `diagrams_tools.py`의 `generate_diagram` 함수가 Unix 전용 `signal.SIGALRM`과 `signal.alarm()`을 사용
- Windows에서 `AttributeError: module 'signal' has no attribute 'SIGALRM'` 오류 발생

### 해결
- Unix 전용 signal 기반 타임아웃을 `concurrent.futures.ThreadPoolExecutor`로 교체
- 크로스 플랫폼 호환성 확보 (Unix/Linux/Windows 모두 지원)

### 변경 사항
**파일: `infrastructure_diagram_mcp_server/diagrams_tools.py`**

1. `import signal` 제거
2. `from concurrent.futures import ThreadPoolExecutor, TimeoutError as FuturesTimeoutError` 추가
3. 별도 스레드에서 다이어그램 코드를 실행하는 `execute_diagram_code()` 내부 함수 추가
4. `ThreadPoolExecutor.submit()` + `future.result(timeout=timeout)`으로 타임아웃 구현

### 테스트
- Python AST 파싱으로 구문 검증 완료
- ThreadPoolExecutor 타임아웃 동작 검증 완료
- pytest 테스트 통과 (3 passed, 7 skipped - Graphviz 미설치로 인한 스킵)

---

## Windows 경로 이스케이프 수정

### 문제
- Windows 경로(`C:\Users\...`)가 Python 코드에 삽입될 때 백슬래시가 이스케이프 시퀀스로 해석됨
- `'unicodeescape' codec can't decode bytes` 오류 발생

### 해결
- `output_path.replace('\\', '/')`로 슬래시로 변환하여 Python 이스케이프 문제 방지

### 변경 사항
**파일: `infrastructure_diagram_mcp_server/diagrams_tools.py`**
- 라인 309-310: Windows 경로 백슬래시를 슬래시로 변환하는 `safe_output_path` 추가

---

## 절대 경로 사용 시 output_dir 미정의 버그 수정

### 문제
- 절대 경로를 filename으로 전달할 때 `output_dir` 변수가 정의되지 않음
- `execute_diagram_code()`에서 `os.chdir(output_dir)` 호출 시 `NameError` 발생

### 해결
- 절대 경로 처리 시 `os.path.dirname()`으로 `output_dir` 추출
- 해당 디렉토리가 없으면 `os.makedirs()`로 생성

### 변경 사항
**파일: `infrastructure_diagram_mcp_server/diagrams_tools.py`**
- 라인 86-89: 절대 경로에서 디렉토리 추출 및 생성 로직 추가

---

## `__main__.py` 모듈 엔트리포인트 추가

### 문제
- `.mcp.json`이 `python -m infrastructure_diagram_mcp_server` 명령어 사용
- 패키지에 `__main__.py` 파일이 없어서 `-m` 옵션으로 실행 불가
- 에러: `No module named infrastructure_diagram_mcp_server.__main__`

### 해결
- `infrastructure_diagram_mcp_server/__main__.py` 파일 생성
- `server.main()` 함수를 호출하는 엔트리포인트 구현

### 변경 사항
**파일: `infrastructure_diagram_mcp_server/__main__.py`** (신규)
```python
from infrastructure_diagram_mcp_server.server import main

if __name__ == '__main__':
    main()
```

---

## MCP 서버 설정 변경

### 변경 사항
**파일: `.mcp.json`**
- `uvx infrastructure-diagram-mcp-server` → `python -m infrastructure_diagram_mcp_server`로 변경
- 로컬 패키지를 사용하여 수정사항 즉시 테스트 가능

### 주의사항
- Claude Code 재시작 필요 (MCP 서버 재로드)

---

## AWS Airflow 배치 처리 아키텍처 다이어그램 생성

### 생성 파일
- `generated-diagrams/aws-airflow-batch-processing.png`

### 아키텍처 구성요소

**오케스트레이션 레이어**
- MWAA (Managed Workflows for Apache Airflow) - 배치 워크플로우 오케스트레이션
- S3 - DAG 저장소

**배치 처리 레이어**
- ECS/Fargate - 장시간 실행 배치 작업 컨테이너
- Lambda - 경량 태스크 실행

**데이터 레이어**
- S3 - 입출력 데이터 저장소, 데이터 레이크
- RDS - 배치 작업 결과 저장소
- Redshift - 분석 데이터 웨어하우스

**메시징 레이어**
- SQS - 배치 작업 큐
- SNS - 작업 완료 알림

**모니터링**
- CloudWatch - 로그, 메트릭, 알람

### 데이터 플로우
1. EventBridge 스케줄 트리거 -> MWAA Airflow
2. Airflow -> ECS Tasks (장시간 배치) / Lambda (경량 태스크)
3. 컴퓨트 -> S3 Data Lake, RDS Results DB
4. 데이터 -> Redshift Analytics
5. ECS -> SQS Job Queue -> SNS Notifications

---

## pygraphviz 및 graphviz2drawio 설치 (Windows)

### 문제
- `.drawio` 파일 변환에 필요한 `graphviz2drawio` 패키지 설치 실패
- `pygraphviz` 빌드 시 `graphviz/cgraph.h` 헤더 파일 찾지 못함

### 해결
1. Graphviz가 이미 설치되어 있음 확인 (`C:\Program Files\Graphviz`)
2. 환경 변수 설정 후 pygraphviz 설치:
```bash
export INCLUDE="C:/Program Files/Graphviz/include"
export LIB="C:/Program Files/Graphviz/lib"
pip install pygraphviz
```
3. graphviz2drawio 설치:
```bash
pip install graphviz2drawio
```

### 결과
- `.drawio` 파일 생성 기능 활성화
- `generated-diagrams/aws-airflow-batch-processing.drawio` 생성 확인
