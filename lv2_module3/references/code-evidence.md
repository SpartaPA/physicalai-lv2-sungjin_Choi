# 분석한 기존 코드의 출처와 조건

- 프로젝트 출처: [Lv2_Monglian_Assignment](https://github.com/Lv2-Monglian-Assignment/Lv2_Monglian_Assignment)
- 작업 저장소 HEAD: `db6e165dad11787feaeb6c03ca9e3b3198276068`. 실행 당시 모듈 3 실험 자료는 미커밋 상태였다.
- 실제 실행 파일: PA18의 `vision_app/v3-runtime-001-prepared/runtime/issue34-v3`에서 복사한 `issue34_live_view.before-tuning.py`와 `target_detector/` 파일.
- 실행본 해시: [source-manifest.json](source-manifest.json). 분석 및 비교 실험 동안 검출 알고리즘은 변경하지 않았다.
- 공개 프로젝트의 HEAD 파일과 준비된 실행본은 바이트가 다르다. 공개 링크는 프로젝트 출처이며, 실제 분석 대상은 아래 해시의 준비된 파일이다.
- 전체 소스 대신 분석에 필요한 조건만 아래에 인용했다. 파일·줄 번호는 준비된 실행본 기준이다.

## `target_detector/detection.py`

SHA256: `5300ef23cb560c9c333e157a240830166abd6dfdb57c628271a7a988114ac839`

99~107행:

```python
def make_mask(hsv, cfg: DetectorConfig, scale=1.0):
    mask = cv2.inRange(hsv, np.array(cfg.hsv_lower, np.uint8), np.array(cfg.hsv_upper, np.uint8))
    if cfg.morph_kernel > 1:
        ks = max(1, int(round(cfg.morph_kernel * scale)))  # 축소 영상에서는 커널도 같은 비율로 줄인다
        ks |= 1                                             # 홀수로 맞춤: 짝수 커널은 중심이 없어 마스크가 한쪽으로 밀린다
        k = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (ks, ks))
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, k)    # 작은 점 잡음 제거
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, k)   # 물체 안의 작은 구멍 메우기
    return mask
```

123~130행:

```python
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    cands = []
    for c in contours:
        if sx != 1.0 or sy != 1.0:
            c = restore_contour(c, sx, sy)
        area = cv2.contourArea(c)
        if area < cfg.min_area_px:
            continue
```

291~295행:

```python
    det.cx, det.cy, det.contour, det.area_px = c.cx, c.cy, c.contour, c.area_px
    det.ex = (c.cx - w / 2) / (w / 2)
    det.ey = (c.cy - h / 2) / (h / 2)
    det.area_ratio = c.area_px / (w * h)
    return det
```

## `target_detector/target_detector.py`

SHA256: `c3a4cbc7838d1bf03652888824fcc8dadb3b1821ab5a8f229e437c1fc6fa9e41`

71~76행:

```python
        self.cfg = DetectorConfig(
            hsv_lower=tuple(p('hsv_lower', [100, 120, 50])),
            hsv_upper=tuple(p('hsv_upper', [130, 255, 255])),
            morph_kernel=int(p('morph_kernel', 5)),
            min_area_px=float(p('min_area_px', 150.0)),
            selection=p('selection', 'largest'),
```

289~295행:

```python
    def publish(self, header, det, dres):
        t = PointStamped()
        t.header = header                                  # 원본 영상 stamp·frame_id 유지
        if det.detected:
            t.point.x, t.point.y, t.point.z = det.ex, det.ey, det.area_ratio
        self.target_pub.publish(t)                         # 미검출이면 (0, 0, 0)

```

## `issue34_live_view.before-tuning.py`

SHA256: `8f67fc467930b0404b8ec3afbf255f594a7109343b6894faa626a3bf8224835d`

25~27행:

```python
        self.overlay_pub = self.create_publisher(CompressedImage, '/target/debug/overlay/compressed', qos)
        self.mask_pub = self.create_publisher(CompressedImage, '/target/debug/mask/compressed', qos)
        self.get_logger().info('LIVE_VIEW_READY /target/debug/overlay/compressed /target/debug/mask/compressed')
```

32~40행:

```python
        for pub, frame in ((self.overlay_pub, draw_overlay(bgr, det, dres, self.last_ids)), (self.mask_pub, mask)):
            ok, jpg = cv2.imencode('.jpg', frame, [cv2.IMWRITE_JPEG_QUALITY, 70])
            if ok:
                msg = CompressedImage()
                if header is not None:
                    msg.header = header
                msg.format = 'jpeg'
                msg.data = jpg.tobytes()
                pub.publish(msg)
```

## 해석

- `make_mask`: HSV 범위를 통과한 영역에 열기·닫기를 적용한다. 닫기로 메워진 픽셀은 원래 HSV 조건을 통과한 픽셀이라고 단정할 수 없다.
- `find_candidates`: Mask에서 찾은 윤곽선의 면적이 `min_area_px`보다 작으면 후보에서 제외한다. 이 조건은 Mask를 지우지 않는다.
- `make_detection`: x와 y는 영상 중심 기준 정규화 좌표, z는 선택한 윤곽선 면적을 영상 전체 면적으로 나눈 값이다.
- `publish`: 미검출이어도 0,0,0 메시지를 발행한다.
- 표시용 실행 파일은 Mask와 검출 표시 영상을 JPEG 품질 70으로 발행한다. 수신 이미지에는 압축에 의한 중간 회색값이 생긴다.
