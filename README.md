✨整體操作流程：
-------------------------
1. 先將 i18n 文件放到對應的資料夾內，此專案僅校驗 messages.po & {language}.json
  分別對應如下：
  i18n_input/{language}/LC_MESSGES/messages.po、
  i18n_input/{language}/{language}.json，
  注意語言名稱必須資料夾與json相同。

2. 執行 "generate_phrase_comparison.py"，會根據資料夾結構，生成一個 "phrase_comparison.xlsx"
  文檔內會有預設的類型與敏感詞（僅作為繁體中文的示例，用戶應自行在各個區域按需填入實際內容）。
  預設的業態類型如果需要維護，需要在 config.yaml 文檔中調整。（不需維護的時候，留空即可）

3. 完成調整後，執行 "script_01_generate_xlsx.py" 
會根據匹配 json 與 po 文件，對應到的 key & value，修改了哪些值、預計修改後的結果，根據語言，在 /i18n_output/ 生成對應的 "{language}_tobemodified.xlsx"
💡注意此處的生成邏輯包含如下：
  a. 一個 value 符合多個敏感詞的情況，將進行替換並標註。
  b. 檢測了存在包含關係的敏感詞替換，並且不會循環替換。
（例如 同時存在 "大學生"替換為"資深員工"、“學生”替換為"人員"，如果 value 中存在"大學生"，僅替換為"資深員工"。）
  c. 檢測後有預計替換的結果將會標黃並說明替換依據，如果空白處想要添加替換結果也可修改。

4. 確認 tobemodified 後，根據需求（生成新的 json/po 或是 部分增補的 json/po），可以分別執行不同 py：

  4-1. 完整生成： "script_02_apply_fixes.py"
    根據修改項進行修改，未修改項略過；重新生成完整的 json/po 文件。

  4-2. 合併現有： "script_02_apply_combine.py"
    需要選擇要合併的檔案，被合併的檔案需要放在 /i18n_combine 下，並自行分別選擇要合併的 tobemodified 語言、
    json/po 檔案名稱，如果只是要產生合併格式的文件，也可以不選擇合併對象；合併時如果存在重複層級的 key & value，會報錯並提示用戶選擇。
    產生的資料夾名稱為 /i18n_output/{language}_時間戳_combined。
    po 文件：根據語言 / 業態逐一產生
    json 文件：根據業態逐一產生，語言會合併在同一文檔

🎉 檔案生成完畢！在 i18n/output/ 中根據時間戳即可即可找到


更新記錄
-------------------------
2025-06-20 多語言敏感詞檢測與替換系統 Version 2.6.2
1. 功能調整：修復當業態中都不存在對應內容的時候，不會產生該業態的合併 json 文件。

2025-06-19 多語言敏感詞檢測與替換系統 Version 2.6.1
1. 功能调整：修復如果是創建新的 json 時，會逐一校驗每個新值對應空值的問題，改為一鍵生成。

2025-06-18 多語言敏感詞檢測與替換系統 Version 2.6
1. 功能調整：由於可能增補的對象不存在現有的 json or po，需要可以自己根據 json(結合語言) 與 對應的 po。
2. 功能調整：修復 json 可以保持多個，但是 po 文檔會被合併成一個，沒有按照語言拆分的問題。
3. 功能調整：修復相同陣列的3個元素如果只被修改第二個，需要抓取對應語言中的 json 文件，在 combine 的結果中完整寫入整個陣列。

2025-06-17 多語言敏感詞檢測與替換系統 Version 2.5
1. 功能調整：由於本身就經常存在相同 key 在不同的層級，故不套用相同 key 在各個層級的校驗。
2. 功能調整：如果不存在對應的 po 文件或是 json 文件，需要按照樣板邏輯生成（json 會合併，po 會按照語言拆分）。


2025-06-10 多語言敏感詞檢測與替換系統 Version 1.0
-------------------------
資料夾：
1. Ori: 手動備份文件，在專案中不會用到。
2. backup：在更新 py, po, json 時，會保存 backup 文件在裡頭，並加上時間戳。
3. __pycache__：我也不知道是啥，Claude 很開心寫的。

檔案，此處僅引用繁體中文的版本：
1. zh-TW.json - 前端文件（內部稱呼）
2. messages.po - 後端文件（內部稱呼）

-
操作流程：
這個專案包含兩套操作，
  1. 敏感詞彙的確認與調整
    1-1. 用戶編輯 phrase_comparison.xlsx，確認後保存。
    1-2. 執行 “phrase_update.py”，會輸入 phrase_comparison.xlsx,
      存在變更時會先備份原本的py文件，然後將新的內容分別寫入
      detection_terms.py,（教育機構）
      detection_terms_enterprises.py,（企業）
      detection_terms_public_sector.py,（公部門）
      detection_terms_training_institutions.py.(培訓機構）
 （*備註：
    這個步驟是可逆的，如果四個py是另外生成的，也可以根據py的內容生成phrase_comparison.xlsx：
    在已經存在這幾個py的情況下，執行"phrase_comparison.py",即可生成 phrase_comparison.xlsx。）

  2. 具體替換敏感詞彙並確認寫入，生成新版本的 json 與 po 文件。
    2-1. 執行整體比對，執行 "script_01_generate_xlsx.py"
      輸出 tobemodified.xlsx 內容會包含如下 
      對比 messages.po, zh-TW.json, 列出內容包含敏感詞彙的 key 與 value
      並且展示對應在企業、公部門、培訓機構的 value 對照，並顯示替換後的 value 值。
    2-2. 用戶確認是否調整（不調整可以將內容改為空），或是整體用另外的方式修改。
      用戶調整完成後，執行 "script_02_apply_fixes.py"
      會將有修改的文件保存到 bakcup 中，並且生成新的對應 po 與 json 文件：
      zh-TW.json
      zh-TW_enterprises.json
      zh-TW_public_sector.json
      zh-TW_training_institutions.json
      messages.po
      messages_enterprises.po
      messages_public_sector.po
      messages_training_institutions.po
