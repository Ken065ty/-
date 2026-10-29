🛡️ EcoGuard AI（綠鏈智安）MVP v1.0
零信任 + AI 異常偵測 + 不可篡改帳本，保護供應鏈碳排數據。
一、安裝（只做一次）
安裝 Python 3.10 以上：https://www.python.org/downloads/ （Windows 安裝時務必勾選 Add Python to PATH）
開啟終端機（Windows：在資料夾空白處按住 Shift + 右鍵 →「在此處開啟 PowerShell」），輸入：
```
pip install -r requirements.txt
python setup_project.py
```
`setup_project.py` 會自動建立供應商、產生 1,000 筆模擬數據、訓練 AI，並顯示攔截率。
二、啟動（需要開兩個視窗）
視窗	Windows	Mac / Linux
① 閘道	雙擊 `start_gateway.bat`	`./start_gateway.sh`
② 戰情室	雙擊 `start_dashboard.bat`	`./start_dashboard.sh`
戰情室會自動開啟瀏覽器（http://localhost:8501）。到「⚔️ 攻擊模擬 Demo」分頁按按鈕，
其他分頁會即時出現結果。閘道的 API 文件在 http://localhost:8000/docs 。
三、自動驗收
```
python -m unittest discover -s tests -v      # 14 項測試，含規格書 TC-01~TC-04
python -m ecoguard.simulate                  # 閘道啟動後，對真實 HTTP 服務跑全部攻擊情境
```
四、與規格書的差異（請務必閱讀）
項目	規格書	本實作	原因
異常分數	原始 Isolation Forest 分數，>0.75 攔截	校準後分數，>0.75 攔截	實測原始分數最高僅約 0.65，直接用 0.75 攔截率不到 5%
特徵	4 個	4 個 + 「每件用電量」	實測可抓到更多「用電量對不上」的造假
Autoencoder	搭配使用	未實作	效益有限，列為第二階段
區塊鏈	Polygon Amoy	預設本機雜湊鏈，Polygon 為選配	上鏈需要你自己的測試錢包，無法代辦
TLS 1.3	模組一包含	需在部署時設定（反向代理）	TLS 屬部署設定，不在應用程式內
Streamlit 即時	—	每 3 秒自動刷新	—
實測 AI 表現（模擬資料）：攔截率約 83%、誤殺率約 1.7%。可調整 `ecoguard/config.py` 的
`ANOMALY_THRESHOLD`：調低 → 攔更多但誤殺增加。
五、選配：上 Polygon Amoy 測試網
建立專用測試錢包（MetaMask 新帳號，絕不要用有真錢的），到水龍頭領免費測試幣 POL。
用 Remix IDE 貼上 `contracts/EcoGuardLedger.sol`，編譯並部署到 Polygon Amoy，複製合約地址。
`pip install web3`，設定環境變數後重啟閘道：
```
POLYGON_RPC_URL=https://rpc-amoy.polygon.technology
POLYGON_PRIVATE_KEY=（測試錢包私鑰）
POLYGON_CONTRACT_ADDRESS=（合約地址）
```
啟動後 `/api/v1/health` 的 `ledger_mode` 會顯示「本機雜湊鏈 + Polygon Amoy」。
上鏈失敗時系統會自動退回本機帳本，不會中斷服務。
六、已知限制（MVP）
防重放快取與速率限制存在記憶體，閘道重啟會清空；多台部署需改用 Redis。
供應商金鑰以明文存在 SQLite（Demo 用）；正式環境應放密鑰管理服務。
模型以模擬數據訓練，正式使用需換成供應商真實歷史資料並重新訓練。
Polygon 上鏈程式碼尚未經實際測試網驗證（沙盒無網路），第一次接上請先小量測試。
檔案地圖
```
ecoguard/security.py   模組一：HMAC、時間戳、重放、限流、IP 白名單
ecoguard/anomaly.py    模組二：Isolation Forest 推論與分數校準
ecoguard/ledger.py     模組三：雜湊鏈 + 選配 Polygon + 驗算器
ecoguard/pipeline.py   串接全流程
ecoguard/main.py       FastAPI 入口
dashboard/app.py       模組四：Streamlit 戰情室
ecoguard/simulate.py   攻擊模擬器
# -
