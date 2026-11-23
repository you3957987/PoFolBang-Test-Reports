# PoFolBang 멀티플레이 검증 보존본

> **공개용 편집본:** 비공개 저장소에서 추출한 보고서입니다. 장비 호스트명과 로컬 실행 경로를 익명화했으며, 원본 코드·로그·추가 파일을 가리키는 내부 링크는 공개본에서 비활성화했습니다. 테스트는 Codex를 활용한 자동화 및 기록을 포함합니다.


2026-11-19 검증에 사용한 리비전은 `3add9910d64147f0aa7d24b77ed67056`이다. Steam 48/48, 에디터 19/19 통과를 원본 결과에서 확인했다. 에디터 결과 19건은 경고를 포함한다. 시험 종료 후 PC와 노트북의 `MultiShooter.exe` 및 Pak·IoStore 5개 파일의 SHA-256이 모두 일치했다. [판정 요약 JSON](Results.json) · 파일별 SHA-256

- 게임 코드·맵 수정과 수정 전후 근거는 최종 보고서 §3, 검증 범위는 §2, 잔여 실패·미검증은 §4에 있다. 최종 검증보고서 · 검증 범위와 남은 조건
- 빌드별 결과와 개별 회차의 PC·노트북 로그 경로는 실행 색인에 있다. 실행 색인 · 거리별 실측 · 20분 연속 경기
- 원본 로그와 캡처 위치는 저장소의 `Saved/TestRuns` 아래 회차별 디렉터리다. 양단 로그는 각 모드·호스트 하위의 `PC.log`와 `Laptop.log`, 실행 결과는 회차 루트의 `results.json`이다. 캡처와 에디터 원본 JSON은 최종 보고서 §8에서 링크한다. Git에서 받은 체크아웃에서 원본 링크까지 열려면 이 경로에 해당 보관본이 있어야 한다. 원본 실행 디렉터리 · 에디터 원본
- `Reports`의 Markdown·요약 JSON·SHA-256 목록은 Git에 추가 가능한 기록이다. `Saved/MPBaseline` 원본 집계, `Saved/TestRuns` 증거 파일, `Saved/StagedBuilds` 패키지는 `Saved/*` 규칙으로 제외된다. Git 제외 규칙

## 다음 검증 실행

저장소 루트의 PowerShell에서 실행한다. UE 5.8.4의 Windows Development 패키지와 서로 다른 Steam 계정 두 개가 필요하다. App ID는 480이다. 노트북 로컬 `[로컬 테스트 도우미 실행 경로]`에 `READY`가 뜨면 기존 공유 폴더와 도우미를 사용한다. 패키지 갱신, `iteration` 배포, 도우미 확인은 기존 실행 안내를 따른다. 양쪽의 배포 리비전과 `MultiShooter.exe`·Pak·IoStore SHA-256을 대조하고 회차를 하나씩 실행한다. 빌드·배포·도우미 설정

```powershell
$runner = '.\Tools\LaptopTest\Run-MultiplayerTests.ps1'
& .\Tools\LaptopTest\Send-LaptopTestRequest.ps1 -Action ping
& $runner -NaturalFlow -HostFPS 60 -ClientFPS 60 -TimeoutSeconds 240
& $runner -Modes FreeForAll -Arsenal -TimeoutSeconds 510
& $runner -Modes FreeForAll -WeaponDamage -TimeoutSeconds 270
& $runner -Modes FreeForAll -RangeTTK -TimeoutSeconds 330
& $runner -Modes FreeForAll -Hosts PC -NaturalFlow -SoakSeconds 1200 -HostFPS 60 -ClientFPS 60 -TimeoutSeconds 1450
```

실행기는 `-gauntlet=PFBMultiplayerTest`를 넘기고 Steam 세션별 고유 방 이름을 생성한다. 기본 호스트는 PC와 노트북이다. 위 명령은 전체 모드 흐름, 총기, 부위별 피해, 거리, 장시간 검사의 재실행 예다. 나머지 F 시나리오와 조건은 [Results.json](Results.json)과 실행 전 판정 기준에서 확인한다. 기능 검사는 기본적으로 `NullRHI`를 사용하며, 화면 증거에는 `-HudProof`, `-TrailShooter Host` 또는 `-TrailShooter Client`처럼 렌더링을 사용하는 회차를 선택한다. 회차 종료 후 `Saved/TestRuns`의 양쪽 로그와 `results.json`을 대조한다.

에디터 회귀는 `PoFolBang.uproject`에서 Session Frontend의 Automation을 열고 `MultiShooter.Combat` 그룹을 실행한다. 기존 실행의 개별 테스트 경로와 경고·오류 개수는 [Results.json](Results.json)에 보존했다.
