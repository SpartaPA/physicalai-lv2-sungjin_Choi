# 모듈 3 — 설정값에 따른 Mask와 검출 결과 변화 분석

작성: 최성진 · Claude Code·Codex와 함께 정리 (2026-10-14)

재실행 방법과 코드 출처는 [README](README.md)에 있다.

## 1. 사용한 환경·입력 사진·토픽 구성

### 1-1. 분석 대상과 자료 출처

기존 프로젝트에서 저장한 사진을 ROS 이미지 토픽으로 다시 입력해, 검출 설정값에 따른 Mask와 검출 결과의 변화를 비교했다. 이 보고서의 실행 기록은 Claude Code·Codex의 자동 실행 보조로 얻은 것이다(저장 사진 입력, ROS 노드 실행, 수신 결과 캡처). 이후의 실험 사실은 이 기록을 기준으로 서술한다.

분석한 코드와 준비된 실행본의 출처는 [코드 출처 기록](references/code-evidence.md)과 [출처 해시](references/source-manifest.json), 실행 결과의 독립 검증은 [검증 기록](references/independent-evidence.json)에 있다.

### 1-2. 목표 물체와 원본 촬영 조건

아래 내용은 [작성자가 알려준 원본 촬영 조건](references/capture-setup.md)이며, 이번 실험에서 측정한 값이 아니다.

- 목표는 원통 2개를 세로로 쌓은 물체다.
- 원통 하나의 규격은 30mm × 30mm × 60mm다.
- 물체의 하단 28mm는 가려진다.
- 원본 촬영은 물체가 원반 위에서 회전하면서 카메라와의 거리가 변하는 조건이었다. 촬영 기록([입력 출처 기록](results/selected-20261014-001/manifest.json))에도 회전판을 켠 카메라 시험이라고 남아 있다. 거리 변화를 직접 재 값은 기록에 없다.
- 위 설명에 따른 산술값으로, 명목 총 높이는 120mm이고 가림을 제외하면 92mm다.

이번 모듈 3의 A/B 비교는 이 원본 촬영의 동적 조건 전체를 다시 검증한 것이 아니다. 원본 촬영 때 저장된 `frame-045` 한 장을 같은 입력으로 반복해 정적 설정만 비교했다. `use_depth=false`라서 거리(m)를 측정한 실험도 아니다. 영상에서의 크기는 픽셀 면적으로만 다루며, 픽셀에서 mm를 거꾸로 계산하지 않았다.

### 1-3. 실행 환경

[환경 기록](results/selected-20261014-001/environment.json): Ubuntu 24.04, Python 3.12.3, OpenCV 4.6.0, ROS 2 lyrical. 이 값은 결과 묶음을 만든 환경에서 수집됐다.

기존 비전 실행 파일(준비된 실행본)과 ROS 노드를 로컬 PC에서 실행했다. [기준 실행의 명령 기록](results/selected-20261014-001/runs/exp1-A/execution.json)에 입력 발행·검출 노드·이미지 뷰어·수신 도구의 실행 명령이 남아 있다. 뷰어는 별도의 가상 화면에서 실행해 캡처했다.

### 1-4. 입력 사진

입력은 원본 시험(182프레임)에서 저장된 사진 3장이다. 사진 크기는 640×480이고, 수신 원본의 색상 인코딩은 `bgr8`이다.

- [목표가 보이는 사진 (frame-045)](results/selected-20261014-001/inputs/present-frame-045.png)
- [목표가 보이지 않는 사진 (frame-170)](results/selected-20261014-001/inputs/absent-frame-170.png)
- [목표 일부가 가려진 사진 (frame-130)](results/selected-20261014-001/inputs/occluded-frame-130.png)

![A/B 비교에 공통으로 사용한 frame-045](results/selected-20261014-001/inputs/present-frame-045.png)

실험 1-A·1-B·2-A·2-B에는 모두 `present-frame-045.png`를 사용했다. [검증 기록](references/independent-evidence.json)에서 네 실행의 수신 원본 픽셀 해시가 같다. 같은 사진을 쓰면 물체의 이동과 조명 변화가 비교에 끼어들지 않고, 설정값 하나만 바꾸면 출력 차이를 그 설정값과 연결해 해석할 수 있다. 목표가 보이지 않는 사진과 일부가 가려진 사진은 기준 설정 A로 한 번씩 돌린 보조 확인이다.

### 1-5. 토픽 구성

기준 실행 1-A의 [토픽 정보](results/selected-20261014-001/runs/exp1-A/received/topic-info.json)에 기록된 구성이다. 임시 노드 이름의 숫자는 실행마다 달라진다.

| 자료 | 토픽 이름 | 메시지 타입 | 발행 노드 | 수신 도구 |
| --- | --- | --- | --- | --- |
| 원본 영상 | `/camera/camera/color/image_raw` | `sensor_msgs/msg/Image` | `/_ros2cli_219736` | RViz, 증거 수신 노드 |
| Mask | `/target/debug/mask/compressed` | `sensor_msgs/msg/CompressedImage` | `/target_detector` | RViz, rqt_image_view, 증거 수신 노드 |
| 검출 표시 영상 | `/target/debug/overlay/compressed` | `sensor_msgs/msg/CompressedImage` | `/target_detector` | RViz, 증거 수신 노드 |
| 검출 수치 | `/target` | `geometry_msgs/msg/PointStamped` | `/target_detector` | 증거 수신 노드 |

[수신 기록](results/selected-20261014-001/runs/exp1-A/received/receipt.json)에 네 토픽의 같은 프레임 수신 결과가 저장되어 있다.

## 2. 설정값 A/B와 바꾸기 전 예상

### 설정과 비교 의도

HSV(에이치에스브이)는 색상 H, 채도 S, 밝기 V로 색을 구분하는 방식이다. 이 8비트 입력에서 V는 0~255 범위를 사용한다. 실험 1은 밝기 V의 하한만 바꿨다. 하한을 높여도 원본 사진이 밝아지지 않고, 색 필터가 더 밝은 픽셀만 받아들이게 된다. 실험 2는 윤곽선 면적의 하한만 바꿨다.

| 실험 | 바꾼 값 | A | B | 고정한 조건 |
| --- | --- | --- | --- | --- |
| 1 — Mask 생성 | `hsv_lower[2]`: V 하한 | 40 | 180 | H 100~130, S 120~255, V 상한 255, 최소 면적 100 px² |
| 2 — 검출 후보 필터 | `min_area_px`: 최소 윤곽선 면적 | 100 px² | 50,000 px² | HSV `[100,120,40]`~`[130,255,255]` |

실험 2는 실험 1-A의 HSV 설정으로 되돌려 진행했다. 네 실행 모두 같은 사진, `detect_scale=1.0`, `morph_kernel=5`, `selection=priority`를 사용했다. 실행에 적용된 `runtime.yaml`에서는 `use_depth=false`, `use_object_tracker=false`다. 원본 설정 파일의 추적 옵션은 true지만 실행 도구가 false로 바꿔 적용했고, 비교에는 실제 적용값을 사용한다. 설정은 노드가 시작할 때 한 번 읽으므로 실행마다 노드를 새로 시작했다.

실제 설정 파일은 [1-A](results/selected-20261014-001/runs/exp1-A/runtime.yaml), [1-B](results/selected-20261014-001/runs/exp1-B/runtime.yaml), [2-A](results/selected-20261014-001/runs/exp2-A/runtime.yaml), [2-B](results/selected-20261014-001/runs/exp2-B/runtime.yaml)의 적용값에서 확인한다.

### 사전 계획에 적힌 예상

[사전 계획 자료](results/selected-20261014-001/pre-run/plan-before-run.md)는 첫 실행 전에 자동 실행 보조가 작성한 기록이다. 이 보고서에서는 사전 계획 자료로만 인용한다.

- 실험 1: 원통 안쪽 일부 영역(12,650픽셀)의 V가 80~121로 측정되어, V 하한을 180으로 올리면 Mask에서 목표가 사라지고 미검출이 될 것으로 예상했다. 이 점검은 원통 전체가 아닌 내부 사각형에 한정됐다.
- 실험 2: 원통 면적을 육안 추정으로 약 1.9만 px²로 보고, 100은 통과하고 50,000은 탈락하되 Mask는 같을 것으로 예상했다.
- 기준 A의 z는 0.05~0.07, 중심 위치는 대략 x=-0.03, y=-0.06으로 예상했다. 별도의 허용 오차는 정하지 않았다.

## 3. 실제 결과와 증거 이미지·로그

### 수신 결과

Mask(마스크)는 처리 후 남은 영역을 보여주는 흑백 영상이다. 아래 Mask 흰색 픽셀 수는 형태학 처리(열기·닫기) 후 JPEG로 압축된 Mask를 수신·복호화해 밝기 127을 초과한 픽셀을 센 값이며, 윤곽선 면적과 다른 값이다. 후보 수는 `/target`의 z가 아니라 검출 로그의 `n_candidates`에서 읽었다.

| 실행 | Mask 흰색 픽셀 | `/target` z | 통과 후보 수 | 결과 |
| --- | ---: | ---: | ---: | --- |
| 실험 1-A (V 40) | 18,979 | 0.060724 | 1 | 검출 |
| 실험 1-B (V 180) | 0 | 0.000000 | 0 | 미검출 |
| 실험 2-A (면적 100) | 18,979 | 0.060724 | 1 | 검출 |
| 실험 2-B (면적 50,000) | 18,979 | 0.000000 | 0 | 미검출 |
| 목표가 보이지 않는 사진 (A) | 0 | 0.000000 | 0 | 미검출 |
| 일부 가려진 사진 (A) | 12,166 | 0.038618 | 1 | 검출 |

### A / 1-B / 2-B Mask 비교

| | 1-A (기준) | 1-B (V 하한 180) | 2-B (최소 면적 50,000) |
| --- | --- | --- | --- |
| 수신 Mask | ![1-A Mask](results/selected-20261014-001/runs/exp1-A/received/mask.png) | ![1-B Mask](results/selected-20261014-001/runs/exp1-B/received/mask.png) | ![2-B Mask](results/selected-20261014-001/runs/exp2-B/received/mask.png) |
| Mask 흰색 픽셀 | 18,979 | 0 | 18,979 |
| 선택된 윤곽선 면적 | 18,654.5 px² | 없음 | 없음 |
| `/target` z | 0.060724 | 0 | 0 |
| 목표가 탈락한 단계 | 탈락 없음 | 색 범위 필터(Mask 생성) | 면적 필터(후보 선택) |

1-B와 2-B는 모두 z=0이지만 Mask가 다르다. 1-B는 Mask 자체가 비었고, 2-B는 1-A와 같은 Mask가 남은 채 후보만 탈락했다.

### 증거 파일

각 실행의 입력 프레임·수신 시각·토픽은 [증거 색인](references/evidence-index.md)에 정리했다.

| 실행 | 수신 영상 | 수치·설정·화면 증거 |
| --- | --- | --- |
| 실험 1-A | [원본](results/selected-20261014-001/runs/exp1-A/received/raw.png) · [Mask](results/selected-20261014-001/runs/exp1-A/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/exp1-A/received/overlay.png) | [수신값](results/selected-20261014-001/runs/exp1-A/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/exp1-A/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/exp1-A/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/exp1-A/rviz.png) |
| 실험 1-B | [원본](results/selected-20261014-001/runs/exp1-B/received/raw.png) · [Mask](results/selected-20261014-001/runs/exp1-B/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/exp1-B/received/overlay.png) | [수신값](results/selected-20261014-001/runs/exp1-B/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/exp1-B/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/exp1-B/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/exp1-B/rviz.png) |
| 실험 2-A | [원본](results/selected-20261014-001/runs/exp2-A/received/raw.png) · [Mask](results/selected-20261014-001/runs/exp2-A/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/exp2-A/received/overlay.png) | [수신값](results/selected-20261014-001/runs/exp2-A/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/exp2-A/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/exp2-A/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/exp2-A/rviz.png) |
| 실험 2-B | [원본](results/selected-20261014-001/runs/exp2-B/received/raw.png) · [Mask](results/selected-20261014-001/runs/exp2-B/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/exp2-B/received/overlay.png) | [수신값](results/selected-20261014-001/runs/exp2-B/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/exp2-B/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/exp2-B/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/exp2-B/rviz.png) |
| 목표가 보이지 않는 사진 | [원본](results/selected-20261014-001/runs/absent/received/raw.png) · [Mask](results/selected-20261014-001/runs/absent/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/absent/received/overlay.png) | [수신값](results/selected-20261014-001/runs/absent/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/absent/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/absent/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/absent/rviz.png) |
| 일부 가려진 사진 | [원본](results/selected-20261014-001/runs/holdout-occluded/received/raw.png) · [Mask](results/selected-20261014-001/runs/holdout-occluded/received/mask.png) · [검출 표시](results/selected-20261014-001/runs/holdout-occluded/received/overlay.png) | [수신값](results/selected-20261014-001/runs/holdout-occluded/received/receipt.json) · [후보 로그](results/selected-20261014-001/runs/holdout-occluded/logs/topic_demo_detect.csv) · [적용값](results/selected-20261014-001/runs/holdout-occluded/runtime.yaml) · [뷰어 캡처](results/selected-20261014-001/runs/holdout-occluded/rviz.png) |

### 결과 읽기

- **실험 1-B:** 원본에는 목표가 있지만 수신 Mask가 비었고 검출값도 0이었다.
- **실험 2-B:** 수신 Mask에는 목표 영역이 그대로 남았는데 검출값만 0이었다.
- **목표가 보이지 않는 사진:** 검출되지 않았다. 물체가 실제 공간에서 치워졌는지는 확인하지 않았다. 입력과 검출 메시지는 계속 수신됐으므로 메시지 중단과는 구분된다.
- **일부 가려진 사진:** 기준 A에서 검출됐다. z는 0.060724에서 0.038618로 작아졌고, 선택된 윤곽선 면적은 18,654.5 px²에서 11,863.5 px²였다. 같은 원본 시험의 다른 프레임에서 영상 속 면적이 달라진 관측이며, 회전·거리·가림 중 무엇이 원인인지는 이 비교로 분리하지 못한다.

기준 A의 수신값은 x=-0.027249, y=-0.044323, z=0.060724였다. z는 사전 예상 범위 0.05~0.07 안이었다. y는 예상 -0.06보다 약 0.0157 큰 값(예상보다 아래쪽)이었고, 이 차이는 허용 오차를 사후에 만들어 통과로 처리하지 않는다. 일부 가려진 사진에서는 후보가 2개 이상일 수 있다고 사전 계획에 적혀 있었지만 실제 통과 후보는 1개였다. 다른 파란 영역이 어느 단계에서 없어졌는지는 확인하지 않았다.

## 4. 왜 그런 결과가 나왔는지 코드 조건으로 설명

코드 인용과 파일별 해시는 [코드 출처 기록](references/code-evidence.md)에 있다. 줄 번호는 준비된 실행본 기준이다.

### 처리 순서

원본 영상 수신 → HSV 변환 → 색 범위 필터 → 잡음 제거·구멍 메우기 → Mask → 윤곽선 면적 필터 → 대상 선택 → 검출 수치·표시 영상 발행

Mask와 검출 결과는 서로 다른 처리 단계의 정보를 보여준다. 이 코드에서 발행하는 Mask는 면적 필터로 탈락한 후보를 지운 영상이 아니다.

### 실험 1: 색 범위 조건

`detection.py`의 `make_mask`(99~107행)는 `cv2.inRange`로 HSV 범위 안의 영역을 만든 다음 열기·닫기로 잡음 제거와 구멍 메우기를 한다. V 하한을 180으로 올리면 V가 180보다 작은 픽셀은 첫 색 범위 검사에서 탈락한다. 사전 점검한 원통 안쪽 영역의 V는 80~121이었고, 실제 수신 결과에서도 1-B의 Mask가 비고 검출값이 0이었다. 이 입력에서는 목표 영역이 Mask 생성 단계에서 유지되지 않았다.

최종 Mask의 흰 영역에는 후처리(닫기)로 메워진 작은 구멍이 포함될 수 있다. 흰 픽셀 하나하나가 HSV 조건을 직접 통과했다고 해석하지 않는다.

### 실험 2: 면적 조건

`find_candidates`(123~130행)는 Mask에서 윤곽선을 찾고, 윤곽선 면적이 `min_area_px`보다 작으면 후보에서 제외한다. 기준 A에서 선택된 윤곽선 면적은 18,654.5 px²로, 최소 면적 100보다 크고 50,000보다 작다. 2-B는 같은 Mask에서 면적 조건만 바꿨으므로 이 후보가 탈락했다. `make_mask`는 `min_area_px`를 사용하지 않기 때문에 수신 Mask는 그대로 남았다.

### 검출 수치의 의미

`make_detection`(291~295행)은 아래처럼 계산한다.

- x = (중심 가로 좌표 - 영상 너비/2) / (영상 너비/2)
- y = (중심 세로 좌표 - 영상 높이/2) / (영상 높이/2)
- z = 선택한 윤곽선 면적 / (영상 너비 × 영상 높이)

x·y·z는 모두 단위가 없는 비율이다. x는 오른쪽일 때, y는 아래쪽일 때 양수다. z는 물체까지의 거리도, 검출 개수도 아니다. 기준 A는 18654.5 / (640 × 480) ≈ 0.060724로 수신값과 일치한다.

`target_detector.py`의 `publish`(289~295행)는 미검출이어도 x=y=z=0인 메시지를 발행한다. 따라서 이 코드에서 "0을 수신했다"와 "메시지가 오지 않았다"는 서로 다른 상태다.

## 5. 최종 선택값과 이유·한계

### 결론

같은 실제 크기의 물체라도 회전, 카메라와의 거리, 가림에 따라 영상에서 차지하는 면적은 달라질 수 있다. 그러므로 하나의 고정 면적 임계값을 모든 환경의 최적값으로 단정할 수 없다. 이는 이번에 새로 측정한 사실이 아니라 영상 면적의 성질에서 나오는 논리적 한계다. 이 보고서에서 면적이 달라진 관측은 일부 가려진 프레임에서 선택된 윤곽선 면적이 18,654.5 px²에서 11,863.5 px²가 된 것뿐이고, 원인은 분리하지 못했다.

이번 고정 사진(frame-045)에서 목표를 보존하는 기준으로 보면, **A(V 하한 40, 최소 면적 100 px²)를 기준값으로 유지할 근거**가 있다. A는 주 실험 사진과 일부 가려진 사진에서 목표를 검출했고 목표가 보이지 않는 사진에서는 검출하지 않았다. V 하한 180과 최소 면적 50,000은 각각 이 사진에서 목표를 놓쳤다. 다만 A가 최적값이라는 뜻은 아니다. 최종 값을 조정하려면 회전 구간, 가림 정도, 거리별 사진으로 추가 비교가 필요하다.

이번 비교에서 가장 분명하게 확인된 것은 두 필터가 서로 다른 단계에서 목표를 놓친다는 점이다.

| | 실패 단계 | Mask | 검출값 |
| --- | --- | --- | --- |
| 1-B (V 하한 180) | 색 범위 필터 | 비어 있음 (흰색 0) | 0 |
| 2-B (최소 면적 50,000) | 면적 필터 | 목표 영역 유지 (흰색 18,979) | 0 |

최종 검출값이 둘 다 0이어도, Mask를 함께 보면 어느 단계에서 탈락했는지 구분할 수 있다.

### 적용 범위와 한계

- 원본은 원반 회전·거리 변화 촬영이지만, 이번 A/B는 그 촬영에서 저장된 `frame-045` 한 장의 정적 설정 비교다. 원본의 동적 조건 전체는 이번에 다시 검증하지 않았다. 동적 조건으로의 일반화는 논리적 한계로만 적었다.
- `use_depth=false`이므로 거리(m)를 측정하지 않았고, z는 면적비다.
- 사진 3장은 같은 시험의 프레임으로 독립된 장면이 아니다. 조명·배경이 다른 환경의 성능까지는 보여주지 않는다.
- 목표가 보이지 않는 사진은 목표가 화면에 보이지 않는 장면이다. 물리적으로 제거됐는지는 확인하지 않았다.
- 현재 자료로 오검출률과 미검출률을 계산하지 않았다. 최소 면적 100은 다른 사진에서 작은 잡음까지 허용할 수 있다.
- 원통 규격, 가림 높이, 원본 촬영 조건은 작성자 설명이며, 이 입력 자료를 제출 과제에 사용해도 되는지와 개인 실행 요건 충족 여부는 이 기록만으로 판단하지 않는다.
- 저장 사진을 ROS 토픽으로 재발행한 입력이며, 사진을 편집하지 않았다.
