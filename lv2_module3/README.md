# 모듈 3 실행 안내

작성: 최성진 · Claude Code·Codex와 함께 정리 (2026-10-14)

분석 보고서는 [report.md](report.md)이다. 프레임 시각과 파일별 토픽 대응은 [증거 색인](references/evidence-index.md)에 있다. 이 문서는 코드 출처, 환경, 입력 조건, 실행 명령과 결과 파일 위치를 정리한다.

## 코드 출처

- 공개 프로젝트: [Lv2_Monglian_Assignment](https://github.com/Lv2-Monglian-Assignment/Lv2_Monglian_Assignment), 작업 저장소 HEAD `db6e165dad11787feaeb6c03ca9e3b3198276068`. 실행 당시 모듈 3 실험 자료는 이 HEAD에 커밋되지 않은 상태였다.
- 실제 실행한 파일은 공개 HEAD의 파일이 아니라 별도로 준비된 실행본이다. 준비된 실행본(`vision_app/v3-runtime-001-prepared/runtime/issue34-v3`)에서 `issue34_live_view.before-tuning.py`와 `target_detector/` 파일을 복사해 사용했다.
- 공개 프로젝트 HEAD의 파일과 이 준비된 실행본은 바이트가 다르다. 분석 대상은 [source-manifest.json](references/source-manifest.json)에 SHA256이 기록된 준비된 파일이다. 분석과 비교 실험 동안 검출 알고리즘은 바꾸지 않았다.
- 주요 파일 해시: `detection.py` `5300ef23…c839`, `target_detector.py` `c3a4cbc7…9e41`, `issue34_live_view.before-tuning.py` `8f67fc46…835d`. 전체 해시와 코드 조건 인용은 [code-evidence.md](references/code-evidence.md)에 있다.

## 실행 환경과 입력

### 실행 환경

[environment.json](results/selected-20261014-001/environment.json) 기록(결과 묶음을 만든 환경에서 수집한 값):

| 항목 | 값 |
| --- | --- |
| OS | Ubuntu 24.04 |
| Python / OpenCV / NumPy / PyYAML | 3.12.3 / 4.6.0 / 1.26.4 / 6.0.1 |
| ROS 2 | lyrical, `rmw_cyclonedds_cpp`, `ROS_DOMAIN_ID=187` |
| 뷰어 | 비공개 가상 화면(Xvfb)과 소프트웨어 렌더링. 바탕화면 창은 조작하지 않았다. |

### 입력 조건

- 입력은 원본 시험에서 저장된 사진 3장이다: [frame-045](results/selected-20261014-001/inputs/present-frame-045.png)(실험 1·2 공통), [frame-170](results/selected-20261014-001/inputs/absent-frame-170.png)(목표가 보이지 않는 사진), [frame-130](results/selected-20261014-001/inputs/occluded-frame-130.png)(일부 가려진 사진). 출처와 해시는 [manifest.json](results/selected-20261014-001/manifest.json)에 있다.
- 사진은 640×480이며, 저장 사진을 `bgr8` ROS 이미지로 `/camera/camera/color/image_raw`에 반복 발행했다. 사진 3장은 같은 시험의 프레임이며 독립된 장면이 아니다.
- 모터와 하드웨어는 쓰지 않았다. `use_depth=false`, `use_object_tracker=false`가 실행에 적용됐다.
- 목표 물체의 규격(원통 2개, 개당 30mm × 30mm × 60mm, 하단 28mm 가려짐)과 원본 촬영 조건(원반 회전, 카메라와 거리 변화)은 작성자 설명이다. 이번 실험은 `frame-045` 한 장의 정적 설정 비교이며 원본의 동적 조건을 다시 검증한 것이 아니다.

## 실행 명령

### 실제 실행 기록

기준 실행 [exp1-A의 execution.json](results/selected-20261014-001/runs/exp1-A/execution.json)에 러너가 시작한 프로세스의 명령이 기록되어 있다. 아래는 그 내용이며, `$WS`는 원래 작업 경로 `/home/pa12/.codex/worktrees/986a/Lv2_Monglian_Assignment`를 줄여 쓴 것이다. Xvfb의 `-displayfd` 번호와 인증 파일 경로는 실행마다 새로 정해진다.

```bash
# 가상 화면
/home/pa12/.cache/module3-display/usr/bin/Xvfb -displayfd 4 -screen 0 1600x1100x24 -screen 1 1200x900x24 \
  -nolisten tcp -auth /tmp/module3-viewers-x_tztbs0/Xauthority -noreset

# 검출 노드 (준비된 실행본 + 적용된 runtime.yaml)
python3 $WS/lv2_module3/demo/20261012-002/source/issue34_live_view.before-tuning.py \
  --ros-args --params-file $WS/lv2_module3/results/selected-20261014-001/runs/exp1-A/runtime.yaml

# 뷰어
rviz2 -d $WS/lv2_module3/results/selected-20261014-001/runs/exp1-A/exp1-A.rviz
ros2 run rqt_image_view rqt_image_view /target/debug/mask/compressed

# 네 토픽의 같은 프레임 수신
python3 $WS/docs/module3/tools/capture_topic_bundle.py \
  --output $WS/lv2_module3/results/selected-20261014-001/runs/exp1-A/received --timeout 20

# 입력 발행 (input.yaml은 입력 사진을 ROS 메시지로 바꾼 파일, 표준 입력으로 전달)
python3 $WS/docs/module3/tools/ros_topic_pub.py topic pub /camera/camera/color/image_raw sensor_msgs/msg/Image \
  --stdin -r 2 --times 20 -w 3 --max-wait-time-secs 15 --keep-alive 1 --print 0
```

위 프로세스들은 러너 `docs/module3/tools/run_bundle_demo.py`가 시작했다. 러너를 호출한 줄 자체는 결과 묶음에 기록되어 있지 않다. 아래 호출 형식은 러너의 인자 정의(`--config`, `--image`, `--output`)와 `execution.json`의 입력 경로에서 구성했다. 여섯 실행의 종료 코드는 모두 0이다([return-codes.tsv](results/selected-20261014-001/runs/return-codes.tsv)).

### 재실행 조건과 방법

**이 결과 저장소만으로는 실행할 수 없다.** 검출 코드와 러너가 이 저장소에 없으므로, 코드가 없는 새 clone에서 아래 명령만으로 재현된다고 할 수 없다. 원래 명령은 다음이 모두 갖춰진 원래 로컬 작업 공간에서만 실행할 수 있다.

- 작업 경로: `/home/pa12/.codex/worktrees/986a/Lv2_Monglian_Assignment`
- 그 안의 러너 `docs/module3/tools/run_bundle_demo.py`와 보조 도구(`capture_topic_bundle.py`, `ros_topic_pub.py`), 준비된 실행본과 `all-images.rviz`가 있는 `lv2_module3/demo/20261012-002/`
- ROS 2 lyrical 환경을 불러온 셸, 그리고 `Xvfb`(또는 `~/.cache/module3-display/usr/bin/Xvfb`), `xauth`, `xdotool`, `rviz2`, `ros2`, `rqt_image_view`, Python의 cv2·numpy·yaml·PIL

러너는 출력 폴더가 이미 있으면 실패하며, 기존 증거를 덮어쓰지 않도록 새 임시 폴더에 출력한다.

```bash
(
cd /home/pa12/.codex/worktrees/986a/Lv2_Monglian_Assignment || exit 1
source /opt/ros/lyrical/setup.bash || exit 1
RUN_ROOT=$(mktemp -d /tmp/module3-rerun-XXXXXXXX) || exit 1
python3 docs/module3/tools/run_bundle_demo.py \
  --config lv2_module3/results/selected-20261014-001/configs/exp1-A/config.yaml \
  --image lv2_module3/results/selected-20261014-001/inputs/present-frame-045.png \
  --output "$RUN_ROOT/exp1-A" || exit 1
printf '재실행 결과: %s\n' "$RUN_ROOT/exp1-A"
)
```

다른 설정은 `--config`만 `exp1-B`, `exp2-A`, `exp2-B`로 바꾸고 `--output`을 새 폴더로 지정한다. `absent`와 `holdout-occluded`는 `exp1-A` 설정에 `--image`만 `absent-frame-170.png`, `occluded-frame-130.png`로 바꾼 실행이다([return-codes.tsv](results/selected-20261014-001/runs/return-codes.tsv) 참조). 제출 전 위 기준 A 명령을 실제로 재실행했고, 종료코드 0과 기존 결과와 동일한 원본 픽셀·Mask·검출값을 확인했다([재실행 확인 기록](references/readme-replay-validation.json)).

## 설정 변경·적용 방법

| 설정 파일 | V 하한 | 최소 면적 |
| --- | ---: | ---: |
| [exp1-A](results/selected-20261014-001/configs/exp1-A/config.yaml), [exp2-A](results/selected-20261014-001/configs/exp2-A/config.yaml) (동일 파일) | 40 | 100 |
| [exp1-B](results/selected-20261014-001/configs/exp1-B/config.yaml) | 180 | 100 |
| [exp2-B](results/selected-20261014-001/configs/exp2-B/config.yaml) | 40 | 50,000 |

러너는 `--config`로 받은 YAML의 `target_detector.ros__parameters`에 `use_object_tracker=false`, `use_depth=false`, `save_every_n=1`, `show_window=false`, `log_dir`, `save_dir`, `run_id=topic_demo`를 덧씌워 출력 폴더에 `runtime.yaml`로 저장한다. 검출 노드는 이 파일을 `--ros-args --params-file`로 받아 시작할 때 한 번 읽는다. 그래서 A/B마다 노드를 새로 시작한다. 각 실행에 실제 적용된 값은 `runs/<실행>/runtime.yaml`에 있다.

## 결과 파일과 보고서

### 제출 파일 구성

```text
lv2_module3/
├── README.md
├── report.md
├── references/
│   ├── code-evidence.md          # 분석한 코드 조건 인용과 해시
│   ├── independent-evidence.json # 수신 결과의 독립 검증 기록
│   ├── source-manifest.json      # 실행본 파일 출처와 SHA256
│   └── evidence-files.json       # 결과 파일 목록과 SHA256
└── results/selected-20261014-001/
    ├── inputs/    configs/    pre-run/    runs/
    └── environment.json  manifest.json  verification.json
```

이 결과 저장소에는 **검출 코드와 실행 도구(러너 등)가 들어 있지 않다.** 소스 코드와 상위 `docs/`는 복사하지 않았다.

### 결과 파일 위치

모두 `results/selected-20261014-001/` 아래에 있다.

| 내용 | 위치 |
| --- | --- |
| 입력 사진 | `inputs/` |
| A/B 설정 원본 | `configs/exp{1,2}-{A,B}/config.yaml` |
| 사전 계획과 사전 점검 | `pre-run/plan-before-run.md`, `pre-run/probe_cylinder_hsv.json` |
| 실행별 결과 | `runs/{exp1-A,exp1-B,exp2-A,exp2-B,absent,holdout-occluded}/` |
| 수신 원본·Mask·검출 표시 영상 | `runs/<실행>/received/{raw,mask,overlay}.png` (`mask.jpg`, `overlay.jpg`는 수신한 JPEG 그대로의 파일) |
| 수신 로그 | `runs/<실행>/received/receipt.json`, `topic-info.json`, `runs/<실행>/logs/topic_demo_detect.csv`, `detector.log`, `publisher.log`, `capture.log`, `node-info.log` |
| 실제 적용 설정 | `runs/<실행>/runtime.yaml` |
| 뷰어 캡처 | `runs/<실행>/rviz.png`, `mask-viewer.png` |
| 실행 명령 기록 | `runs/<실행>/execution.json` |
| 환경·입력 출처·검증 | `environment.json`, `manifest.json`, `verification.json`, `../../references/independent-evidence.json` |
| 파일 해시 목록 | `../../references/evidence-files.json` |

### 이 자료가 보여주는 범위

- `frame-045` 한 장으로 A/B 설정을 비교한 결과이며 원본 촬영의 회전·거리 변화를 다시 검증한 것이 아니다.
- 결과 수치는 [report.md](report.md)에 정리했다. 최적값이나 과제 전체의 완료를 주장하지 않는다.
- 이 입력 자료의 사용 허용 여부와 개인 실행 요건은 이 기록만으로 판단하지 않는다.
