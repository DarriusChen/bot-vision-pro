# 餐桌視覺開發計畫

第一版用固定俯拍的手機在 Mac 上即時辨識湯匙和叉子，並把辨識到的中心點換成桌面座標。桌面上的物品先分開放置，且靠近桌面平面。餐巾紙、抓取點和手臂整合接在定位驗證之後。

## 專案結構

| 路徑 | 用途 |
| --- | --- |
| `demo.py`（預計新增） | 啟動即時畫面，依序擷取、辨識、換算座標並顯示。 |
| `vision/` | 放相機讀取、模型推論和桌面定位程式。 |
| `scripts/` | 放桌面校正工具。 |
| `calibration/` | 放相機與桌面的校正結果。 |
| `data/demo/` | 放本機測試照片或短片，供重現問題。 |
| `outputs/`（執行時建立） | 放定位 JSON 和示範截圖。 |

## 1. 手機即時辨識

1. 把手機固定在桌面上方，讓畫面涵蓋手臂預定的作業範圍。iPhone 可用 Mac 的接續互通相機，支援 USB 網路攝影機模式的 Android 手機可用 USB 接入。關掉人物居中等自動取景功能。 [Apple 接續互通相機](https://support.apple.com/en-gb/guide/mac-help/mchl77879b8a/mac) · [Google Pixel 網路攝影機模式](https://support.google.com/pixelcamera/answer/14274129)
2. 用 README 的 YOLO 指令確認手機畫面可讀，並試辨識湯匙與叉子。COCO 預訓練模型支援這兩個類別。 [Ultralytics 即時影像輸入](https://docs.ultralytics.com/modes/predict/#inference-sources) · [COCO 類別表](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/datasets/coco.yaml)
3. 將影像讀取放在 `vision/camera.py`，模型推論放在 `vision/detector.py`。預計用 `python demo.py --camera 1` 啟動，索引依電腦辨識到的裝置調整。

**驗收方式**是確認手機即時影像可讀，且移動桌上的餐具時，遮罩跟著更新。

## 2. 桌面校正與即時座標

1. 在桌面上貼至少四個不共線的記號，分布在作業區周圍，用尺量出它們相對於桌面原點的座標。第一版可手點記號，之後再視需要改用 ArUco。
2. 實作 `scripts/calibrate_table.py`，用直播畫面中的像素座標與桌面座標求出 homography。需要做鏡頭去畸變時，校正和推論應使用同一套去畸變流程。將矩陣、直播影像尺寸和桌面座標定義記到 `calibration/phone_table.yaml`。相機移動、換鏡頭或影像裁切改變時重新校正。 [OpenCV 相機校正](https://docs.opencv.org/4.x/dc/dbb/tutorial_py_calibration.html) · [OpenCV Homography](https://docs.opencv.org/4.x/d9/dab/tutorial_homography.html)
3. 在 `vision/localization.py` 將餐具遮罩中心轉為桌面公尺座標，在即時畫面上疊出類別和 `(x, y)`。

**驗收時**先用未參與校正的桌面點量誤差，再用不同位置的餐具測整套流程。記錄桌面中央和邊緣的誤差，和手臂組依夾爪及目標物尺寸訂容許範圍。

## 3. 手臂介面與後續辨識

- 先用 JSON 讓手臂組測試，至少包含類別、辨識信心、擷取時間、`frame_id`、`x_m`、`y_m` 和 `point_type`。桌面原點、正軸方向及 `table` 到手臂底座的轉換由雙方實測確認。如果手臂採用 ROS 2，可用 `tf2` 管理座標轉換。 [ROS 2 tf2](https://github.com/ros2/geometry2)
- 首版的 `point_type` 為 `center`，表示物品中心。需要夾餐具柄時，加上柄的位置標註，輸出抓取候選點和所需的平面角度。第一次聯調先檢查手臂在桌面上方是否對準，再測下探與夾取。
- 加入餐巾紙時，用實際桌面和相機收集影像並標註遮罩，涵蓋不同折法、光線和桌面材質。依拍攝場次切分訓練與測試資料。 [Ultralytics 遮罩標註格式](https://docs.ultralytics.com/datasets/segment)
