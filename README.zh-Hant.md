# 九巴到站時間顯示板（KMB ETA）

適用於 [ESP32-2432S028R](https://github.com/witnessmenow/ESP32-Cheap-Yellow-Display)（俗稱 Cheap Yellow Display／CYD）的桌上顯示板：左半顯示香港天文台天氣，右半顯示最多四條[九巴](https://www.kmb.hk/)路線的預計到站時間。

English version: [README.md](README.md)

## 功能一覽

- 即時氣溫、當日最高／最低溫，以及日／夜天氣背景
- 最多同時顯示 4 條九巴路線 ETA（分鐘）
- 首次開機 Wi‑Fi 設定熱點（`ESP32 Wifi`）
- 瀏覽器設定頁：路線、天氣觀測站、螢幕亮度
- 可選 3D 列印外殼：`kmb-eta 2.8_ box.stl`

## 所需器材

| 項目 | 說明 |
| --- | --- |
| ESP32-2432S028R（2.8" CYD） | ILI9341 螢幕，附 microSD 卡槽 |
| microSD 卡 | FAT32 格式，4 GB 或以上即可 |
| USB 線 | 需支援傳輸資料（用來燒錄） |
| Wi‑Fi | 只支援 2.4 GHz（ESP32 不支援 5 GHz） |
| PlatformIO | 指令列（`pio`）或 PlatformIO IDE |

## 專案結構

```text
KMB--ETA-/
├── platformio.ini          # CYD 板型與 TFT_eSPI 腳位
├── Arduino                 # 原始 Arduino 草圖（保留作參考）
├── src/main.cpp            # PlatformIO 實際編譯的韌體
├── *.bmp                   # 畫面圖檔（需複製到 SD）
├── test_*.vlw              # 平滑字型（需複製到 SD）
└── kmb-eta 2.8_ box.stl    # 可選外殼
```

## 準備 SD 卡

韌體會從 SD 卡**根目錄**讀取所有圖片與字型。請將卡格式化為 **FAT32**，然後把下列檔案複製到根目錄（不要放進子資料夾）：

| 檔案 | 用途 |
| --- | --- |
| `kmb.bmp` | 全螢幕開機／底圖（320×240） |
| `day.bmp` | 日間左側背景（166×240） |
| `night.bmp` | 夜間左側背景 |
| `cloud.bmp` | 日間多雲左側背景 |
| `night_cloud.bmp` | 夜間多雲左側背景 |
| `test_3.vlw`、`test_8.vlw`、`test_12.vlw`、`test_16.vlw`、`test_26.vlw`、`test_32.vlw`、`test_45.vlw` | 畫面使用的字型 |
| 其他 `test_*.vlw` | 專案內額外字型；全部複製亦可 |

請在開機**前**插入 SD 卡。若 SD 無法掛載，螢幕會變紅，序列埠會顯示 `SD Fail`。

## 編譯與燒錄

1. 安裝 [PlatformIO Core](https://docs.platformio.org/en/latest/core/installation.html)。
2. 用 USB 連接 CYD。
3. 若序列埠不是 `/dev/cu.usbserial-2110`，請修改 `platformio.ini` 的 `upload_port`／`monitor_port`（Windows 多為 `COM…`，Linux 常見 `/dev/ttyUSB0`）。
4. 在專案根目錄執行：

```bash
pio run -e cyd -t upload
```

如需完整清除（會刪除已存 Wi‑Fi 與路線設定）：

```bash
pio run -e cyd -t erase
pio run -e cyd -t upload
```

開啟序列埠監控：

```bash
pio device monitor -b 115200
```

## 首次設定 Wi‑Fi

1. 插入已準備好的 SD 卡後開機。
2. 螢幕應先變白，再顯示九巴開機圖。
3. 用手機或電腦連接開放熱點 **`ESP32 Wifi`**。
4. 開啟設定頁（或前往 `http://192.168.4.1`）。
5. 選擇家中 2.4 GHz Wi‑Fi 並輸入密碼。
6. 連線成功後，顯示板會開始更新天氣與巴士時間。

之後若要重設 Wi‑Fi，可到設定頁按 **重設 WiFi**，或按上文清除 flash。

## 日常使用指南

### 開啟設定頁

CYD 已連上家中網絡後：

- 建議使用：[http://KMB-ETA.local](http://KMB-ETA.local)
- 或改用路由器／序列埠顯示的 IP 位址

### 設定天氣觀測地區

1. 開啟設定頁。
2. 在 **天氣觀測地區** 選擇天文台測站（預設：黃大仙）。
3. 按 **儲存並更新**。

### 新增巴士路線（最多 4 條）

1. 輸入路線號碼，例如 `243M`。
2. 選擇方向：去程／回程。
3. 如有需要，選擇服務類型（常規 `1` 或特別班次 `2`）。
4. 等待車站清單載入，再選取車站。
5. 按 **＋ 新增至清單**。
6. 可重複新增其他路線。
7. 按 **儲存並更新** — 裝置會重啟並顯示新清單。

### 調整亮度

在設定頁使用亮度滑桿，範圍約 `10` 至 `255`。設定會寫入 flash 保存。

### 畫面說明

| 區域 | 內容 |
| --- | --- |
| 左側 | 時間、日期、目前氣溫、當日最高／最低溫、天氣背景 |
| 右側 | 最多四列巴士：路線、目的地、到站分鐘 |

ETA 顯示 `0`／`-` 代表即將到站；`!` 代表暫時沒有 ETA。

已連網時，大約每 5 秒更新一次。

## 可選外殼

可列印 `kmb-eta 2.8_ box.stl`，尺寸對應 2.8 吋 CYD。

## 疑難排解

| 狀況 | 建議處理 |
| --- | --- |
| 紅畫面／出現 `SD Fail` | 重新插卡；確認 FAT32，且檔案在根目錄 |
| 出現白畫面與九巴圖後停住 | 連接 `ESP32 Wifi` 並完成 Wi‑Fi 設定 |
| 完全黑屏 | 檢查 SD 檔案；燒錄最新韌體；背光為 GPIO 21、高電位開啟 |
| 沒有天氣／巴士資料 | 檢查上網；確認可連天文台／九巴 API |
| 顏色錯亂或畫面花 | 確認板子是 `platformio.ini` 所用的 2.8" ILI9341 CYD 腳位 |
| 找不到 `KMB-ETA.local` | 改用 IP；部分手機不支援 `.local`／mDNS |
| 找不到燒錄埠 | 更新 `platformio.ini` 的 `upload_port` |

## 資料來源

- 天氣：[香港天文台開放數據](https://data.weather.gov.hk/)
- 巴士 ETA：[九巴／龍運 ETA API](https://data.etabus.gov.hk/)

本專案為非官方作品，與九巴、天文台無關。

## 授權／來源說明

韌體以本倉庫原始 Arduino 草圖為基礎。建置設定對應社群常用的 CYD（ESP32-2432S028R）腳位。
