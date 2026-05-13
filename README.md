## 發送機制

1.  **第一條訊息必須是 Template**：這點完全正確。只要是你在過去 24 小時內沒有和該客戶互動過，第一條訊息就**必須**是經過 Meta 審核的 Template。
2.  **不用等客戶回覆，你就能發第二條**：你**不需要**苦苦等待客戶回覆才能發第二條訊息。只要你高興，你可以立刻連續發送第二條、第三條訊息。
3.  **關鍵限制：在客戶回覆前，你的每一條訊息都必須是 Template**。


## 用實際場景舉例：

❌ 錯誤的做法（會被 Meta API 報錯拒絕）

-   **第 1 條**：發送 Template（出貨通知） ➔ **成功**
-   **第 2 條**：立刻發送自定義純文字：「對了，出貨單號是 12345 喔！」 ➔ ❌ **失敗 (報錯)**
    -   _原因：客戶還沒回覆，這時還沒開啟「24小時對話視窗」，不能發送自由文字。_

綠色 正確的做法（連續發送 Template）

-   **第 1 條**：發送 Template A（出貨通知） ➔ **成功**
-   **第 2 條**：立刻發送 Template B（邀請評價，裡面帶有自定義變數） ➔ **成功**
    -   _原因：雖然客戶沒回覆，但你第二條也是用合規的範本，所以 Meta 允許發送。_

綠色 客戶回覆後的做法（解鎖自由對話）

-   **第 1 條**：你發送 Template A（出貨通知） ➔ **成功**
-   **客戶回覆**：「收到，謝謝！」 ➔ 🌟 **解鎖 24 小時自由對話視窗**
-   **第 2 條**：你發送完全自定義的文字：「不客氣！祝您使用愉快 🌟」 ➔ **成功**
-   **第 3 條**：你發送完全自定義的圖片（產品說明書） ➔ **成功**

---

## 建立這種高自由度的範本

### 步驟一：在 Meta 開發者後台建立「萬用範本」

請前往 Meta 商業管理員的 WhatsApp 訊息範本管理後台，建立一個新範本：

1.  **類別**：選擇「行銷（Marketing）」或「公用程式（Utility）」。
2.  **名稱**：設定為 `flexible_notification`。
3.  **語言**：選擇 `繁體中文 (zh_HK)` 或 `zh_TW`。
4.  **內容欄位 (Body)**：不要寫死任何字，直接填入變數：
    
    > `{{1}}`
    
    _(或者寫成這樣更像親切的通知：`你好 {{1}}！{{2}}`)_
5.  **按鈕欄位 (Buttons)**：選擇「快速回覆 (Quick Reply)」，設定一個按鈕文字，例如：
    
    > `聯絡真人客服`
    
提交審核後，通常 2-5 分鐘內就會自動通過。


### 步驟二：使用 PHP 發送「萬用範本 + 動態純文字 + 按鈕」

當你需要主動傳訊給客戶時，你可以把**整段想說的話**當作一個長字串，直接塞進 `{{2}}` 變數裡。

以下是完整的 PHP cURL 實作程式碼：

    <?php
    $accessToken = 'YOUR_ACCESS_TOKEN';
    $phoneNumberId = 'YOUR_PHONE_NUMBER_ID';
    $url = "facebook.com{$phoneNumberId}/messages";
    
    // 你想對客戶說的完全自定義內容（包含換行、表情符號）
    $customMessage = "感謝您在我們網站下單！\n\n您的訂單編號為 #HK99823。\n商品預計將於 2-3 個工作天內送達。\n\n如果有任何問題，請點擊下方按鈕與我們聯絡！";
    
    $data = [
        'messaging_product' => 'whatsapp',
        'to' => '85291234567', // 接收者手機號碼
        'type' => 'template',
        'template' => [
            'name' => 'flexible_notification', // 剛才審核通過的萬用範本名稱
            'language' => [
                'code' => 'zh_HK'
            ],
            'components' => [
                // 替換 Body 欄位中的動態變數
                [
                    'type' => 'body',
                    'parameters' => [
                        [
                            'type' => 'text',
                            'text' => '張先生' // 對應 {{1}}（客戶姓名）
                        ],
                        [
                            'type' => 'text',
                            'text' => $customMessage // 對應 {{2}}（整段自定義內文）
                        ]
                    ]
                ]
            ]
        ]
    ];
    
    // cURL 發送流程
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        'Authorization: Bearer ' . $accessToken,
        'Content-Type: application/json'
    ]);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    
    if (curl_errno($ch)) {
        echo '錯誤: ' . curl_error($ch);
    } else {
        echo "狀態碼: {$httpCode}\n";
        echo "回應結果: {$response}\n";
    }
    
    curl_close($ch);
    ?>
