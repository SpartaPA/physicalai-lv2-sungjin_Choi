# 수신 증거 찾아보기

각 실행 폴더 이름은 실험 번호와 A/B 조건을 구분한다. 아래 시각은 저장 사진을 다시 발행했을 때의 ROS 메시지 시각이며 원본 촬영 시각이 아니다. 원본 촬영 시각은 manifest.json의 original_metadata에 있다.

| 실행 폴더 | 입력 파일 | 수신 프레임 시각(sec.nanosec) | 같은 프레임의 묶음 |
| --- | --- | --- | --- |
| `exp1-A` | `present-frame-045.png` | `1791971163.282329822` | [수신 묶음](../results/selected-20261014-001/runs/exp1-A/received/receipt.json) |
| `exp1-B` | `present-frame-045.png` | `1791971184.836085345` | [수신 묶음](../results/selected-20261014-001/runs/exp1-B/received/receipt.json) |
| `exp2-A` | `present-frame-045.png` | `1791971207.072791700` | [수신 묶음](../results/selected-20261014-001/runs/exp2-A/received/receipt.json) |
| `exp2-B` | `present-frame-045.png` | `1791972438.075100695` | [수신 묶음](../results/selected-20261014-001/runs/exp2-B/received/receipt.json) |
| `absent` | `absent-frame-170.png` | `1791971251.329328648` | [수신 묶음](../results/selected-20261014-001/runs/absent/received/receipt.json) |
| `holdout-occluded` | `occluded-frame-130.png` | `1791971274.559192538` | [수신 묶음](../results/selected-20261014-001/runs/holdout-occluded/received/receipt.json) |

프레임 식별자: `camera_color_optical_frame`.

| 폴더 안의 파일 | 토픽 / 의미 |
| --- | --- |
| `received/raw.png` | `/camera/camera/color/image_raw` 원본 수신 영상 |
| `received/mask.png` 및 `mask.jpg` | `/target/debug/mask/compressed` 복호화 영상 및 원래 수신 JPEG |
| `received/overlay.png` 및 `overlay.jpg` | `/target/debug/overlay/compressed` 검출 표시 영상 및 원래 수신 JPEG |
| `received/receipt.json` | 같은 프레임에서 받은 `/target` x/y/z, 토픽명, 영상 정보 |
| `received/topic-info.json` | 실제 발행·구독 노드와 메시지 타입 |
| `runtime.yaml` | 실행에 적용한 설정값 |
| `logs/topic_demo_detect.csv` | 프레임별 검출 여부, 후보 수, 중심 좌표, 처리 기록 |
| `rviz.png` / `mask-viewer.png` | 실제 ROS 이미지 뷰어의 수신 화면 |
| `execution.json` | 입력·실행 명령·종료 상태·뷰어 실행 방식 |

대표 캡처는 별도의 가상 화면에서 실행된 실제 RViz/rqt의 화면이다. 바탕화면에서 사용자가 직접 창을 조작한 증거로 해석하지 않는다. 정식 여섯 실행의 결과만 포함하며 중간 시도·중복 캡처·도구 전체 코드는 제출본에서 제외했다.
