# 모듈 1

라즈베리파이 Ubuntu 22.04에서 OpenCR에 위치 P 제어 펌웨어를 업로드하고, 문제 1을 실행한 기록이다. 문제 2~4는 아직 없다.

## 장비·환경

- 호스트 `pa18`, Ubuntu 22.04.5 LTS, aarch64
- OpenCR R1.0, `/dev/ttyACM0`, USB `0483:5740`
- XM430-W350, ID 12, 1 Mbps, Protocol 2.0
- 컴파일러 `arm-none-eabi-g++ 10.3.1`, Arduino CLI 1.5.1, 업로더 `opencr_ld` 1.0.4

환경 명령의 출력은 [results/환경확인.txt](results/환경확인.txt)에 있다.

## 실행 방법

1. 라즈베리파이에서 만든 `opencr_position_p.ino.bin`을 OpenCR에 업로드한다. 업로드 중에는 모터 커넥터를 빼 둔다.
2. 모터를 고정하고 12V를 켠 뒤, 고장 상태가 아닌지 확인한다.
3. `s 1 10 10`으로 한 번 실행한다. Kp는 1, 속도 상한은 10 deg/s, 상대 목표각은 10 deg이다.
4. 3.1초에 `x`로 멈춘다.

설명은 [report.md](report.md)에 있다.

## 결과 파일

- [results/환경확인.txt](results/환경확인.txt): 호스트 이름, Ubuntu 버전, 아키텍처, 포트
- [results/upload.log](results/upload.log): `CRC OK`, `[OK] Download`
- [results/실행A.log](results/실행A.log): 목표 변경 이후 12행, 마지막 줄 `STOP: user`

이번 커밋은 문제 1만 포함한다. 모듈 전체 완료 태그는 문제 4까지 끝난 뒤에 붙인다.
