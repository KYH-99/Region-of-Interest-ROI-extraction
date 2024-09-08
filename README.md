# Region-of-Interest-ROI-extraction Public
2024 | Open CV Term Project

# 영상처리 (Open CV) - 이미지 관심 영역(ROI) 추출하기

## 프로젝트 개요

이 프로젝트는 OpenCV를 활용하여 이미지에서 관심 영역(Region of Interest, ROI)을 추출하는 기능을 구현합니다. 사용자는 타원 또는 다각형 형태로 영역을 지정할 수 있으며, 해당 영역을 추출하여 저장할 수 있습니다.

## 사용된 도구 및 플랫폼

- **Python**: 메인 프로그래밍 언어
- **OpenCV**: 이미지 처리 라이브러리
- **Numpy**: 수학 연산 및 배열 처리를 위해 사용
- **PyCharm**: 개발 환경

## 주요 기능

1. **타원 영역 추출**:
   - 사용자가 마우스를 드래그하여 타원형 관심 영역을 지정할 수 있습니다.
   - 지정된 영역은 추출되어 이미지 파일로 저장됩니다.

2. **다각형 영역 추출**:
   - 사용자가 여러 개의 점을 찍어 다각형 영역을 설정할 수 있습니다.
   - 다각형 내부의 영역을 추출하여 이미지로 저장합니다.

3. **모드 전환**:
   - 타원 모드와 다각형 모드를 키보드를 통해 전환할 수 있습니다.
   - ESC 키를 누르면 프로그램이 종료됩니다.

## 결과 화면

### (1) 타원 모드에서의 관심 영역 지정

사용자가 마우스를 드래그하여 타원 영역을 설정할 수 있습니다. 드래그 중에는 실시간으로 타원이 그려지고, 마우스를 놓으면 최종적으로 타원이 설정됩니다.

### (2) 다각형 모드에서의 관심 영역 지정

다각형 모드에서는 여러 점을 클릭하여 다각형을 설정합니다. 점과 점 사이에 선이 그려지며, 첫 번째 점과 가까운 거리를 클릭하면 다각형이 닫히고 해당 영역이 추출됩니다.

### (3) 추출된 이미지 저장

설정한 타원 또는 다각형 영역이 이미지로 추출되어 저장됩니다.

## 핵심 코드

```python
import cv2
import numpy as np

drawing = False
mode = 'NULL'
ix, iy = -1, -1
img_count = 1
Polygon_points = []

def onMouse(event, x, y, flags, param):
    global ix, iy, drawing, mode, image, Polygon_points, img_count

    if event == cv2.EVENT_LBUTTONDOWN:
        if mode == 'Ellipse':
            drawing = True
            ix, iy = x, y

        elif mode == 'Polygon':
            if len(Polygon_points) > 0 and np.linalg.norm(np.array(Polygon_points[0]) - np.array([x, y])) < 20:
                Polygon_points.append(Polygon_points[0])
                cv2.polylines(image, [np.array(Polygon_points)], isClosed=True, color=(0, 255, 0), thickness=2)
                save_roi_polygon()
                Polygon_points.clear()
            else:
                Polygon_points.append((x, y))
                if len(Polygon_points) > 1:
                    cv2.line(image, Polygon_points[-2], Polygon_points[-1], (0, 255, 0), 2)
                cv2.circle(image, (x, y), 3, (0, 0, 255), -1)
            cv2.imshow('image', image)

    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing and mode == 'Ellipse':
            temp_image = image.copy()
            radius = int(np.sqrt((x - ix) ** 2 + (y - iy) ** 2))
            cv2.circle(temp_image, (ix, iy), radius, (255, 0, 0), 2)
            cv2.imshow('image', temp_image)

    elif event == cv2.EVENT_LBUTTONUP:
        if drawing and mode == 'Ellipse':
            drawing = False
            radius = int(np.sqrt((x - ix) ** 2 + (y - iy) ** 2))
            cv2.circle(image, (ix, iy), radius, (0, 255, 0), 2)
            save_roi_ellipse(ix, iy, radius)
            cv2.imshow('image', image)

def save_roi_ellipse(ix, iy, radius):
    global img_count
    x1, y1 = max(0, ix - radius), max(0, iy - radius)
    x2, y2 = min(image.shape[1], ix + radius), min(image.shape[0], iy + radius)
    roi = image[y1:y2, x1:x2].copy()
    cv2.imwrite(f'roi_{img_count:04d}.jpg', roi)
    img_count += 1

def save_roi_polygon():
    global img_count
    mask = np.zeros_like(image)
    cv2.fillPoly(mask, [np.array(Polygon_points)], (255, 255, 255))
    masked_image = cv2.bitwise_and(image, mask)
    x, y, w, h = cv2.boundingRect(np.array(Polygon_points))
    roi = masked_image[y:y + h, x:x + w].copy()
    cv2.imwrite(f'roi_{img_count:04d}.jpg', roi)
    img_count += 1

def switch(key):
    global mode, Polygon_points
    if key == ord('1'):
        mode = 'Ellipse' if mode != 'Ellipse' else 'NULL'
    elif key == ord('2'):
        mode = 'Polygon' if mode != 'Polygon' else 'NULL'
        if mode == 'NULL' and len(Polygon_points) > 2:
            cv2.polylines(image, [np.array(Polygon_points)], isClosed=True, color=(0, 255, 0), thickness=2)
            Polygon_points.clear()
    print(f'Mode switched to: {mode}')

image = cv2.imread('old.jpg')
cv2.imshow('image', image)
cv2.setMouseCallback('image', onMouse)

while True:
    key = cv2.waitKey(1) & 0xFF
    if key == 27:
        break
    switch(key)

cv2.destroyAllWindows()
```

## 태그

- Python
- OpenCV
- 이미지 처리
- 관심 영역(ROI) 추출
- 영상 처리
- 타원 및 다각형
