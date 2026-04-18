# 行銷專案管理系統 — 部署資訊

## 基本資訊
- **系統名稱**：行銷專案管理系統
- **類型**：HTML 單檔（類型 A）
- **部署平台**：GitHub Pages
- **線上網址**：https://jacky5453-beep.github.io/marketing-project-manager/
- **GitHub Repo**：https://github.com/jacky5453-beep/marketing-project-manager
- **最後部署日期**：2026-04-18

## Firebase 設定
- **Firebase 專案**：`project-management-69f5b`（獨立於共用的 `derlife-audit`）
- **認證方式**：Google Sign-In
- **Firestore 結構**：
  - `users/{uid}/data/main` — 主要資料（projects、tasks）
  - `users/{uid}/backups/{timestamp}` — 版本快照（最多保留 20 份，每 5 分鐘寫一次）

## 部署指令
```bash
cd "/Users/jacky/Desktop/claude/claude code/專案管理/deploy"
git add -A
git commit -m "描述"
git push origin main
# GitHub Actions 自動部署 → 1~2 分鐘後線上版更新
```

## 環境變數
無（Firebase config 直接寫在 index.html 前端，這是公開資訊可被客戶端看到）

## 資料保護機制（2026-04-18 新增）

系統有以下多重保護，避免資料意外遺失：

1. **dataLoaded 保護旗標**：Firestore 尚未成功載入前，`save()` 一律略過
2. **讀取失敗即停**：`loadFromFirestore()` 失敗會拋錯；`onAuthStateChanged` 會攔截並跳提示，禁止後續自動存檔
3. **localStorage 備份**：每次 `save()` 同時寫入 `localStorage['mpm_v2_backup']`，即使 Firestore 壞掉本地還有一份
4. **Firestore 版本快照**：每 5 分鐘最多寫入一份快照到 `users/{uid}/backups/`，保留最新 20 份
5. **Console 還原指令**：
   - `listSnapshots()` — 列出雲端最近 20 份快照
   - `restoreSnapshot('快照ID')` — 還原指定快照
   - `restoreLocalBackup()` — 從本機 localStorage 還原

## 歷史事件
- **2026-04-18**：發現並修復資料覆蓋 bug。舊版程式因「靜默讀取失敗」+「登入即自動 save」的組合，會在 Firestore 讀取失敗時把初始 demo 資料覆蓋到雲端，造成使用者資料遺失。Commit d13dcb1 修復。
