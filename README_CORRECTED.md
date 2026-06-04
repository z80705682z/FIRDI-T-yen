# 營養標示自動試算與合規檢驗儀表板 (Nutrition Label Dashboard)

本專案是一個符合台灣衛福部 (MOHW / FDA) 法規的**包裝食品營養標示自動試算與標籤產生器**。使用者可輸入配方原料克數與成品熟重，系統將會自動進行合規性稽核、計算營養密度，並產生符合台灣標準雙框格式的營養標示貼紙，支援下載為 PNG 圖片與複製 HTML 代碼。

---

## 🔗 系統流程圖 (GitHub 支援自動渲染)

GitHub 會自動將下方的 Mermaid 語法渲染成精美的網頁流程圖：

```mermaid
flowchart TD
    classDef startEnd fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e293b;
    classDef process fill:#f1f5f9,stroke:#64748b,stroke-width:1px,color:#0f172a;
    classDef check fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f;
    classDef export fill:#ecfdf5,stroke:#059669,stroke-width:2px,color:#065f46;

    Start([開始試算]) --> Step1[步驟一：輸入原料與重量]
    
    subgraph InputPhase["1. 配方輸入與參數設定"]
        Step1 --> InputSearch{搜尋原料資料庫}
        InputSearch -->|存在於預設庫| SelectDB[點選原料並輸入克數]
        InputSearch -->|不存在於預設庫| AddCustom[新增自訂原料並輸入數值]
        SelectDB --> List[加入配方清單]
        AddCustom --> List
        List --> Step2[步驟二：設定包裝與熟化參數]
        Step2 --> SetCooked[輸入成品總重量]
        Step2 --> SetServings[輸入每一份量及本包裝含份數]
    end

    SetCooked --> ClickFinish{點擊結束配方輸入}
    
    subgraph CalcPhase["2. 核心密度計算與損耗調整"]
        ClickFinish --> SumRaw[加總配方各原料營養素]
        SumRaw --> AdjLoss[計算損耗調整：每100g熟重密度]
    end

    AdjLoss --> Step3[步驟三：法規合規性稽核]

    subgraph AuditPhase["3. 中華民國衛福部法規檢驗"]
        Step3 --> Aud1{成品熟重合理？}
        Aud1 -->|否| Warn1[警告：可能加水未申報]
        Aud1 -->|是| Aud2{"份量 x 份數等於成品重？"}
        
        Aud2 -->|否| Warn2[錯誤：份量份數邏輯不符]
        Aud2 -->|是| Aud3{"碳水是否大於等於糖 + 纖維？"}
        
        Aud3 -->|否| Warn3[錯誤：申報數值衝突]
        Aud3 -->|是| AudPass[稽核通過 / 生成報告]
    end

    Warn1 --> Aud2
    Warn2 --> Aud3
    Warn3 --> AudPass

    AudPass --> Step4[步驟四：修約與0界值處理]

    subgraph RoundingPhase["4. 台灣官方修約與零值轉換"]
        Step4 --> R_Sodium["鈉含量：四捨五入至整數，小於等於5mg標示0"]
        Step4 --> R_Cal["熱量：小於等於4kcal標示0"]
        Step4 --> R_Fat["反式脂肪：小於等於0.3g標示0"]
        Step4 --> R_Sat["飽和脂肪：小於等於0.1g標示0"]
        Step4 --> R_Other["蛋白質、脂肪、碳水、糖、纖維：四捨五入至小數一位，小於0.5g標示0"]
    end

    R_Sodium --> OutputFormat
    R_Cal --> OutputFormat
    R_Fat --> OutputFormat
    R_Sat --> OutputFormat
    R_Other --> OutputFormat

    subgraph OutputPhase["5. 格式化輸出與匯出"]
        OutputFormat["依規定順序排列表格：<br/>1.熱量 2.蛋白質 3.脂肪<br/>(含飽和與反式) 4.碳水化合物<br/>(含糖與膳食纖維) 5.鈉"] --> RenderUI[產生標準黑白雙框標籤貼紙]
        RenderUI --> DrawChart[繪製三大營養素熱量圓餅圖]
        RenderUI --> ExportPng[下載高解析度標籤 PNG 貼紙]
        RenderUI --> CopyHtml["複製網頁用 HTML / CSS 程式碼"]
    end

    ExportPng --> EndNode([完成標籤製作])
    CopyHtml --> EndNode

    class Start,EndNode startEnd;
    class Step1,Step2,Step3,Step4,List,SelectDB,AddCustom,SetCooked,SetServings,SumRaw,AdjLoss,R_Sodium,R_Cal,R_Fat,R_Sat,R_Other,RenderUI,DrawChart process;
    class InputSearch,ClickFinish,Aud1,Aud2,Aud3 check;
    class ExportPng,CopyHtml export;
```