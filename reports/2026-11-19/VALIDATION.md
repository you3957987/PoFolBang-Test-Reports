# Validation — 2026-11-17

> **공개용 편집본:** 비공개 저장소에서 추출한 보고서입니다. 장비 호스트명과 로컬 실행 경로를 익명화했으며, 원본 코드·로그·추가 파일을 가리키는 내부 링크는 공개본에서 비활성화했습니다. 테스트는 Codex를 활용한 자동화 및 기록을 포함합니다.


Environment: desktop [TEST_DESKTOP], laptop [TEST_LAPTOP], separate logged-in Steam accounts, UE 5.8.4, Development package, Steam App ID 480. Laptop helper runs as the normal interactive user through the existing PoPolBang share.

## Two-machine session/map checks

All six combinations passed with deployed revision `116593ac464d48229b6c8f29acbf0c02`:

| Mode | Host | Result | Seconds |
| --- | --- | --- | --- |
| FreeForAll | PC | Pass | 51.7 |
| FreeForAll | Laptop | Pass | 53.0 |
| Team | PC | Pass | 49.6 |
| Team | Laptop | Pass | 50.9 |
| TeamFlag | PC | Pass | 48.2 |
| TeamFlag | Laptop | Pass | 51.5 |

Evidence under the repository:

- `Saved/TestRuns/20261117-191405-b5bcd0e6/results.json`: FreeForAll, PC host.
- `Saved/TestRuns/20261117-191528-e2722bc7/results.json`: FreeForAll, laptop host.
- `Saved/TestRuns/20261117-191633-f4d3c0f2/results.json`: Team and TeamFlag, both hosts.
- Each case has PC and laptop logs with the same unique run ID and both `PFBTEST PASS` markers.

After the final versioned full cook and deployment, TeamFlag with the laptop hosting passed again in 51.5 seconds on revision `059fe8707d16427ea42828803644b293`. Evidence: `Saved/TestRuns/20261117-192604-671dab7a/results.json`. The other five cases above were run before this packaging-setting change, with the same test/gameplay implementation.

Assertions: existing Steam session create/find/join succeeds; normal lobby travels to the selected map; both machines observe two player states, two characters with player states, and their local possessed character for eight seconds in the expected network role. This does not validate damage across machines, flag pickup/capture/score replication, UI clicks, or visual correctness.

## Helper and editor checks

- 28 local helper integration checks passed in real Windows PowerShell 5.1, including file-lock recovery, atomic JSON replacement, arguments, current-session/expiry enforcement, process ownership, deployment locks, changed files, obsolete files, and no-op transfers. Evidence: `Saved/LaptopTestValidation/6b228603a7ed4ae4b83d9f8ed2af5830`.
- Corrected the observed status-write crash: atomic replacement and compatible read sharing replace Move-Item overwrite; transient heartbeat locks no longer terminate the agent. The fixed files were hash-verified on the laptop and exercised throughout the real matrix.
- Corrected the initial Gauntlet class lookup crash by invoking `-gauntlet=PFBMultiplayerTest`.
- Unreal MCP initialized on loopback port 8000; toolset discovery, schema discovery, test discovery, and execution worked. `.codex/config.toml` contains the project connection.
- All 9 existing `MultiShooter.Combat.*` editor regression tests passed, zero failed/skipped. Mesh/socket warnings from test fixtures remain. Evidence: `Saved/SetupArtifacts/combat-results.json`. These are separate from real two-machine combat assertions.
- The project skill passed `quick_validate.py` validation.

## Build and diagnostics

- Incremental build measured 37.9 seconds after enabling the editor plugins; a cached run with the versioned-content flag measured 24.1 seconds. Times depend on changed code/assets and cache state.
- UE reported that legacy iterative cooking with unversioned content can miss C++ serialization changes. The Development build script now uses `-VersionCookedContent`; a one-time full cook rebuilt all 1,492 packages successfully in 38.6 seconds, with zero cook warnings/errors. Evidence: `Saved/Logs/LaptopTest-VersionedFullCook.log`.
- Live Coding in the agent-started editor initially blocked packaging. The owned editor was closed cleanly before rebuilding. Do not force-close an unrelated editor.
- The final changed package deployed in 50.0 seconds. An immediately repeated deployment copied no files and completed in 2.8 seconds (robocopy code 2: excluded metadata present). Evidence: `Saved/SetupArtifacts/incremental-publish.json` and `Saved/SetupArtifacts/unchanged-publish.json`. A changed packed container may still require copying the whole container; this is file-level incremental transfer, not binary delta patching.
- PC/laptop trace files were captured for both FreeForAll host directions. Unreal Insights parsed the PC trace and exited 0 after analysis. Evidence: `Saved/SetupArtifacts/Insights-validation.log`. The diagnostic trace contains incomplete final-frame/memory-profiler warnings after process termination; it is not a clean memory-profiling validation or an assertion of replication correctness.

Generated evidence is under ignored `Saved/` paths and is local to this checkout. Keep this report's scope distinct from future gameplay tests.

## 2026-11-18 노트북 성능 측정 및 반복 작업 정리

- RTX 4070 Laptop GPU에서 패키지를 1280×720 창 모드, VSync 해제, FPS 제한 해제로 측정했다. 각 장면은 20초 워밍업 후 22초의 유효 표본을 사용했다. 기본 메인 메뉴와 움직이지 않는 1인 깃발전 장면이다.
- 수정 전 메뉴 평균 6.169ms / p95 7.213ms, 수정 후 6.093ms / p95 6.904ms였다. 깃발전은 수정 전 6.962ms, 수정 후 6.990ms로 거의 같았다. 정지 장면의 뚜렷한 FPS 개선은 확인되지 않았다. 정확한 수치는 아래 원본 JSON을 따른다.
- 메뉴의 활성 Niagara는 0개, 깃발전은 1개였다. 깃발전에서 기존의 활성 효과를 잠시 숨기고 멈춘 비교 구간은 약 0.25~0.28ms 빨랐다. 이 표본에서 이펙트가 전체 부하의 주원인이라는 증거는 없다.
- 메뉴와 클라이언트의 불필요한 지연 보상 기록 Tick을 중지했다. 서버의 경기 중 기록은 유지한다. 파티원 이름표는 내용 변경 시 다시 그리도록 했다. 피격 파티클과 획득 Niagara는 완료 후 자동 반환하는 컴포넌트 풀을 사용한다.
- 같은 무기 피격 효과를 초당 약 50개 요청하는 비교에서 고유 컴포넌트는 1,070개에서 50개로 감소했다. 평균 프레임은 7.254ms 대 7.307ms로 비슷했다. 프로세스 사용 메모리 표본은 1,446.4MB 대 1,624.4MB로 증가했고, 메모리 절감은 입증하지 않았다. 두 구간은 같은 프로세스에서 순서대로 측정했으므로 메모리 차이를 풀 단독 비용으로 해석하지 않는다.
- 기준값은 `Saved/TestRuns/Performance-Before-7492bec1d2054ead83e13e53979bfbfd/results.json`, 수정 후 정지 장면은 `Saved/TestRuns/Performance-After-3bfabcb01e314d26912f09bf31dc227f/results.json`, 반복 효과는 `Saved/TestRuns/Performance-After-522893468a65401fb6d2abecfda055b4/TeamFlagGameMap.log`에 있다.
- 배포 revision `c6a94b02759d4ab79d182d80c185d8fb`의 고핑 이동 표적 피격은 편도 150ms 조건에서 양쪽 피해 10으로 일치했다. `Saved/TestRuns/20261118-100130-92ca9295/results.json`에 기록했다. PC와 노트북 호스트 깃발 획득·득점·복귀는 `Saved/TestRuns/20261118-100243-bd578508/results.json`에서 모두 통과했다. 파티 양방향 외형·닉네임 갱신과 캡처는 `Saved/TestRuns/SteamParty-d07809d674304344876c5818223dfd30/results.json`에서 통과했다.
- 8인 실제 전투, 장시간 누적 부하, 노트북 기본 해상도에서의 성능은 이 측정에 포함되지 않는다.

## 게임플레이 검사 추가

### 2026-11-18 설정 확장 편집기 검사

- 패키징 없이 MultiShooterEditor Development를 컴파일하고 편집기의 게임 실행 모드로 검사했다. 실행 창은 오프스크린으로 렌더링했다.
- 기존 메인 메뉴 설정 버튼으로 6개 탭이 열리고 정상 복귀했다. 감도·FOV·음량·조준 방식·정보 표시 저장 후 파일 재로드, Enhanced Input의 점프 키 Space→K 변경, 15초 화면 변경 자동 복구, 기본값 초기화가 통과했다.
- 설정값과 화면 모드 검사는 전용 SaveGame 및 GameUserSettings 파일에서 수행했다. 기존 사용자 저장 파일은 시험에 사용하지 않았다. 시험 저장 폴더는 각 결과 폴더의 RuntimeSaved에 보관했다.
- 최종 결과 및 실제 화면 캡처는 `Saved/TestRuns/PFBSettingsProof-46d6ad233e144570ac69f12aee1d365f/results.json`에 있다. PC·노트북 패키지 실행, 실제 오디오 청취 및 모든 키 조합의 수동 플레이 검사는 이 범위에 포함되지 않는다.

- RustDesk 없이 PC와 노트북에서 실행한다. PC는 숨김 창으로 패키징하고 게임은 기본적으로 렌더링 없이 실행한다.
- PC 호스트 깃발전에서 획득, 득점 1점, 원래 위치로 복귀를 양쪽 로그로 확인했다. 증거는 `Saved/TestRuns/20261117-195303-2e7a67b0/results.json`과 같은 폴더의 PC·노트북 로그에 있다.
- PC 호스트 일반전에서 조준과 발사를 실행하고 탄약 소모를 검사했다. 피격된 노트북 플레이어의 체력과 실드 합계가 100에서 90으로 감소한 결과를 양쪽 로그에서 확인했다. 증거는 `Saved/TestRuns/20261117-195540-48153d78/results.json`과 같은 폴더의 PC·노트북 로그에 있다.
