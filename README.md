# Strata 설치·활용 가이드 (RTX 5090 노트북)

> 원본 저장소: <https://github.com/Niko1221/Strata> (MIT License)
> 이 문서는 저장소 README를 바탕으로 정리한 개인 작업 노트입니다. 세부 옵션은 저장소의 `docs/` 문서를 확인하세요.

## 목차

1. [Strata란](#1-strata란)
2. [사전 점검](#2-사전-점검)
3. [설치](#3-설치)
4. [모델 크기 선택](#4-모델-크기-선택)
5. [사용법](#5-사용법)
6. [성능 측정과 튜닝](#6-성능-측정과-튜닝)
7. [문제 해결](#7-문제-해결)
8. [노트북 운용 팁](#8-노트북-운용-팁)
9. [내 환경 기록](#9-내-환경-기록)

---

## 1. Strata란

- **Qwen3.8-Flash-Next**(약 1250억 파라미터급 대형 모델)를 일반 게이밍 PC에서 돌리는 오픈소스 추론 엔진입니다.
- 모델은 24,576개의 작은 전문가(expert) 묶음 구조이며, 토큰마다 10개만 사용합니다.
- 자주 쓰는 전문가는 **GPU VRAM**, 전체 전문가는 **RAM**, 나머지 연산은 **CPU**, 대용량 조회 테이블은 **SSD**에 둡니다.
- 챗, 코딩, 이미지 입력을 지원하고 모든 데이터가 PC 밖으로 나가지 않습니다.
- 로컬에서 **OpenAI / Anthropic 호환 API**를 제공합니다.

## 2. 사전 점검

| 항목 | 요구 사항 |
|---|---|
| GPU | NVIDIA RTX 20/30/40/50 시리즈, **VRAM 12GB 이상** |
| RAM | **32GB 이상** (RAM이 선택 가능한 모델 크기를 결정, 64GB면 모든 크기 가능) |
| 디스크 | 여유 공간 약 **80GB**, SSD 권장 (첫 시작이 훨씬 빠름) |
| OS | Windows 10/11 또는 Linux |
| 드라이버 | 최신 NVIDIA 그래픽 드라이버 |

- [ ] NVIDIA 드라이버 최신 버전 설치
- [ ] 노트북 RAM 용량 확인: `____ GB`
- [ ] 디스크 여유 공간 80GB 이상 확인
- [ ] 전원 어댑터 연결, 고성능 전원 모드 설정

## 3. 설치

### 3-1. Windows (직접 설치)

1. 저장소를 내려받습니다.
   ```bash
   git clone https://github.com/Niko1221/Strata.git
   cd Strata
   ```
   (또는 GitHub의 zip을 받아 압축 해제)
2. **`START-HERE.bat`** 을 더블클릭합니다.
3. 설치 마법사가 GPU를 자동 인식하고 다음을 묻습니다.
   - 어떤 모델, 어떤 크기로 할지
   - 컨텍스트 길이(모델이 한 번에 기억하는 텍스트 양)
   - 이미지 입력 지원 여부
4. 처음에는 **Enter**로 권장값을 선택합니다.
5. 모델(약 70GB)을 내려받고 시작하면 브라우저가 `http://127.0.0.1:8080` 을 엽니다.
6. 다운로드가 중간에 끊기면 같은 파일을 다시 실행합니다(이어받기).

### 3-2. Linux

```bash
git clone https://github.com/Niko1221/Strata.git
cd Strata
./setup.sh
```

### 3-3. AI 코딩 도구에게 맡기기

Claude Code, Cursor, Codex, GitHub Copilot 등에 아래 문장을 붙여 넣습니다. GPU, RAM, 디스크를 점검해 맞는 모델을 고르고 설치·시작까지 진행해 줍니다.

```text
Set up Strata on this PC for me: https://github.com/Niko1221/Strata - follow docs/AI_SETUP.md in that repository.
```

AI 도구가 MCP 서버를 통해 Strata를 설치·시작·중지하는 방법은 저장소의 `docs/MCP_SERVER.md`를 참고하세요.

### 3-4. 첫 실행 시 주의

- 모델을 올리는 **1~3분 동안 PC가 느려지거나 멈춘 것처럼** 보일 수 있습니다(첫 실행이 가장 오래 걸림).
- Strata가 35~55GB를 RAM에 올리고 일부를 GPU용으로 고정(lock)하기 때문이며 정상입니다.
- 창을 닫지 말고 기다리세요. 10분 넘게 멈춰 있으면 PC 재시작 후 다른 프로그램을 닫고 더 작은 크기로 다시 시도합니다.

## 4. 모델 크기 선택

같은 모델을 압축률만 달리한 여러 크기가 있습니다. 작을수록 빠르고, 클수록 조금 더 똑똑합니다.

| 노트북 RAM | 추천 |
|---|---|
| 32GB | **Coder** (코딩 특화). 24GB 카드면 Q2_0, IQ2_XS도 가능 |
| 48GB | **IQ2_XS** (가장 빠른 것은 Q2_0) |
| 64GB | **IQ2_XS** 권장, 여유가 있으면 IQ3_XXS / IQ3_S (IQ3_S가 가장 똑똑하고 가장 느림) |
| 96GB 이상 | IQ3_S 또는 Unsloth UD-IQ4_XS (약 4비트) |

선택 시 참고할 점은 다음과 같습니다.

- **Coder**는 전문가를 절반 제거한 코딩용 버전입니다. 코드 외 영역, 특히 한국어·중국어·일본어(CJK)에서는 약하므로 한국어 문서 작업에는 **Q2_0 / IQ2_XS / IQ3_S** 를 권장합니다.
- **Swift 1.5**: 답하기 전 생각 시간이 훨씬 짧은 파인튜닝판입니다.
- **Unsloth UD-IQ4_XS 및 그 이상(UD-Q4_K_XL ~ Q8_0)**: 품질은 높지만 RAM이 부족하면 SSD에서 읽어 와서 크게 느려집니다(64GB PC에서 UD-Q4_K_XL이 초당 7~8.5토큰).
- 모델을 추가하려면 `SETUP.bat` (Linux: `./setup.sh --setup`)을 실행합니다.
- 크기별 상세는 저장소 `docs/MODELS.md`를 참고하세요.

### 참고 속도 (README 측정값, RTX 5070 12GB / 64GB RAM)

| 크기 | 답변 생성 | 프롬프트 읽기 |
|---|---|---|
| Q2_0 | 94 tok/s | 2,650 tok/s |
| IQ2_XS | 79 tok/s | 2,090 tok/s |
| IQ3_XXS | 62 tok/s | 1,750 tok/s |
| IQ3_S | 53 tok/s | 1,620 tok/s |
| Coder | 55 tok/s | 2,180 tok/s |

VRAM이 더 큰 카드는 더 빠르며, 저장소는 24GB 카드(RTX 3090)가 초당 약 100~140토큰일 것으로 예상합니다. 5090 노트북 실측값은 [9. 내 환경 기록](#9-내-환경-기록)에 남기세요.

## 5. 사용법

### 5-1. 브라우저

`http://127.0.0.1:8080` 에 접속합니다.

| 탭 | 기능 |
|---|---|
| Chat | 대화 화면 |
| Monitor | 모델 상태와 GPU/CPU/RAM 실시간 모니터 |
| About | 설정과 접속 주소 |

### 5-2. 생각(Thinking) 수준

채팅 메뉴 또는 앱의 "reasoning effort"에서 **off / low / medium / high** 를 고릅니다. off가 가장 빠르고, 어려운 문제에는 high가 좋습니다.

### 5-3. 이미지 입력

설치 시 `Images?` 질문에 yes를 선택했다면, 채팅의 **Picture** 버튼으로 첨부하거나 사용하는 앱에서 이미지를 첨부합니다.

### 5-4. 다른 앱·코딩 에이전트 연결

| 용도 | 설정 |
|---|---|
| OpenAI 호환 앱 | Base URL `http://127.0.0.1:8080/v1` (API 키, 모델명은 아무 값) |
| Anthropic API 사용 앱 | `http://127.0.0.1:8080/v1/messages` |
| Claude Code | 환경변수 `ANTHROPIC_BASE_URL=http://127.0.0.1:8080` |
| Codex CLI 등 (Responses API) | `/v1/responses` (설정법은 `docs/DETAILS.md`) |

간단한 호출 예시 (OpenAI 호환):

```bash
curl http://127.0.0.1:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer anything" \
  -d '{
    "model": "strata",
    "messages": [{"role": "user", "content": "안녕하세요, 자기소개를 해주세요."}]
  }'
```

### 5-5. 다른 PC·폰에서 접속

```bat
START-HERE.bat --setup --host 0.0.0.0 --api-key <비밀키>
```

외부 접속을 열 때는 **반드시 API 키를 설정**합니다.

### 5-6. 동작 특성

- 기본은 **한 번에 요청 하나**만 처리하고 나머지는 대기합니다. 동시 처리는 설정에 `"parallel": 2` 를 지정합니다(`docs/BATCHING.md`). 12GB 카드에서는 개별 응답이 느려집니다.
- 채팅의 **첫 메시지는 전체를 읽느라** 약 3만 토큰에 1분이 걸리고, 후속 메시지는 몇 초 만에 시작합니다.

### 5-7. 실행·종료·업데이트

| 작업 | Windows | Linux |
|---|---|---|
| 시작 / 재시작 | `START-HERE.bat` | `./setup.sh` |
| 종료 | 실행 창 닫기 | 실행 창 닫기 |
| 업데이트(실행 안 함) | `UPDATE.bat` | `./update.sh` |
| 모델 추가 | `SETUP.bat` | `./setup.sh --setup` |
| 벤치마크 | `BENCH.bat` | `./bench.sh` |

## 6. 성능 측정과 튜닝

### 6-1. 벤치마크

`BENCH.bat` (Linux: `./bench.sh`) 를 실행하면 5~10분 동안 채팅, 4K/16K 프롬프트, 에이전트식 세션을 측정하고 공유용 리포트를 만들어 줍니다.

### 6-2. 선택 튜닝 (GPU 1장 + 모델 전체가 RAM에 들어갈 때)

데이터 폴더의 `strata-<모델>.json` 에 다음을 추가하고 Strata를 재시작합니다. 되돌리려면 두 항목을 삭제합니다.

```json
"args": [ ..., "--adapt-async", "1" ],
"env": { "STRATA_PIN_BY_NEED": "1" }
```

| 옵션 | 효과 (README, RTX 5070 / Windows / Q2_0 기준) |
|---|---|
| `--adapt-async 1` | 생성 속도 +4~5%, 코드·긴 문서에서 +7~10%. 답변이 기본값과 미세하게 달라질 수 있음 |
| `STRATA_PIN_BY_NEED=1` | Windows에서 프롬프트 읽기 약 +4.4% |

## 7. 문제 해결

| 증상 | 대처 |
|---|---|
| 처음 실행에서 PC가 멈춤 | 정상. 1~3분 기다리기 (10분 넘으면 재시작 후 작은 크기로) |
| 다운로드·설치 중 중단 | `START-HERE.bat` 다시 실행 (이어받기) |
| 매우 느리고 디스크 불빛이 계속 깜빡임 / "the engine stopped unexpectedly" | RAM 부족. 다른 프로그램(특히 브라우저) 종료 또는 Q2_0 / IQ2_XS로 변경 |
| 포트 8080이 이미 사용 중 | 이미 Strata가 실행 중이므로 해당 창 확인 |

- 상세: 저장소 `docs/TROUBLESHOOTING.md`
- 해결되지 않으면 GitHub Issues에 `strata-<모델>.log` 를 첨부해 문의합니다.
- 보안 문제는 `SECURITY.md` 절차에 따라 비공개로 제보합니다.

## 8. 노트북 운용 팁

> 아래는 저장소 README 내용이 아니라 노트북 사용을 위한 개인 메모입니다.

- 장시간 추론은 부하가 크므로 **전원 어댑터 연결** 및 고성능 전원 모드를 사용합니다.
- 제조사 제어 앱에서 성능 모드와 GPU 모드(dGPU 전용 등)를 확인합니다.
- 쿨링 패드 등으로 열을 관리해 쓰로틀링을 줄입니다.
- 첫 로딩은 RAM을 많이 쓰므로 브라우저와 무거운 프로그램을 닫고 시작합니다.

## 9. 내 환경 기록

| 항목 | 값 |
|---|---|
| GPU | RTX 5090 Laptop |
| VRAM | |
| RAM | |
| 저장장치 | |
| OS / 드라이버 버전 | |
| 선택한 모델 크기 | |
| 컨텍스트 길이 | |
| Strata 엔진 버전 | |

### 벤치마크 결과 (`BENCH.bat`)

| 날짜 | 모델 크기 | 답변 생성 (tok/s) | 프롬프트 읽기 (tok/s) | 비고 |
|---|---|---|---|---|
| | | | | |

### 메모

- 

---

## 참고 링크

- 저장소: <https://github.com/Niko1221/Strata>
- 모델: <https://huggingface.co/Qwen/Qwen3.8-Flash-Next>
- 상세 문서: `docs/INSTALL.md`, `docs/MODELS.md`, `docs/DETAILS.md`, `docs/TROUBLESHOOTING.md`, `docs/COMMUNITY_BENCHMARKS.md`
