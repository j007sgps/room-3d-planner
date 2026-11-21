# Room 3D Planner

一套把真實租屋／買房物件「從捲尺量測、格局照片一路還原成可互動 3D 模擬器」的完整實戰套件（含誤差校正、React Three Fiber 3D 空間建模、預覽防誤觸鎖與手機版第一人稱虛擬搖桿漫遊）。

## 安裝方式

### 1. Antigravity 使用者（推薦）

直接將本倉庫 Clone 到全域 Plugin 目錄即可自動載入：

**Windows (PowerShell)：**

```powershell
git clone https://github.com/j007sgps/room-3d-planner.git "$env:USERPROFILE\.gemini\config\plugins\room-3d-planner"
```

**macOS / Linux：**

```bash
git clone https://github.com/j007sgps/room-3d-planner.git ~/.gemini/config/plugins/room-3d-planner
```

安裝後即可在 Antigravity 的 Customizations 頁面看到 **Room 3D Planner** 卡片，並從下一輪對話起直接呼叫使用。

---

### 2. Claude Code / Cursor / Gemini CLI 等其他 AI 工具使用者

只要將本套件中的 `skills/room-3d-planner/` 子資料夾複製到對應工具的 `skills/` 目錄（或專案根目錄的 `.agents/skills/room-3d-planner/`）即可：

```bash
git clone --depth 1 https://github.com/j007sgps/room-3d-planner.git /tmp/room-3d-planner
cp -r /tmp/room-3d-planner/skills/room-3d-planner ~/.agents/skills/room-3d-planner
rm -rf /tmp/room-3d-planner
```

---

## 包含內容

- `plugin.json`：Antigravity Plugin 描述檔與 3 組開箱範例提示詞
- `assets/logo.svg`：高對比透視空間 SVG 圖示
- `skills/room-3d-planner/`：核心建模 Skill
  - `SKILL.md`：四階段閘門式工作流程（確保先校正尺寸，再動手建模）
  - `references/measurement-cleaning.md`：捲尺閉合誤差修正、正交對齊、異常值偵測與日系／實地看房常見歧義排查
  - `references/r3f-architecture.md`：`cm` 單一真相座標、家具衝突與四向距離標尺、50 步 Undo/Redo、預覽／編輯鎖定模式 (`P`)、手機響應式與第一人稱雙搖桿觸控架構
  - `references/r3f-pitfalls.md`：Three.js / Drei 實戰 9 大防雷清單（正交相機文字爆版、Z-fighting、`pointercancel`、CJK 工具列換行等）

## 建議起手提示詞

1. 「幫我為新物件建立 3D 室內空間與家具規劃模擬器，先從 Phase 1 搜尋格局圖開始」
2. 「這是我用捲尺量的房間尺寸與照片，請先幫我做閉合誤差修正與平面確認表」
3. 「幫現有的 3D 房間模擬器補上預覽／編輯鎖定模式與手機版第一人稱虛擬搖桿」

## License

MIT License — 詳見 [LICENSE](./LICENSE)。
