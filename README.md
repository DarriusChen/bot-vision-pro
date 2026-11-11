# Bot Vision Pro

用手機相機先做餐桌物品的即時辨識與桌面定位，後續將定位結果交給機器手臂。

開發順序、驗收方式和預計的座標介面見 [開發計畫](docs/plan.md)。

## 試跑即時辨識

先讓 Mac 把手機辨識為網路攝影機，再於專案根目錄執行以下指令。

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
yolo segment predict model=yolo26n-seg.pt source=0 classes=42,44 show=True save=False
```

`source=0` 是相機索引。如果畫面來自 Mac 內建鏡頭，改用手機對應的索引，例如 `source=1`。第一次執行需要下載模型權重。
