- バックテスト完了後に**1クリックで「公開レポートURL」発行**
    
    - Next.jsで`/r/[slug]`をISR配信、**OG画像（エクイティ/メトリクス）**を自動生成（satori/OG Image）。
        
    - CTAは**「この戦略をコピーして検証」**。未ログインはサインアップ→**戦略JSONを自動インポート**して即実行。
        
- **トークン設計**
    
    - R2: `strategies/{uuid}.json`（条件・期間・銘柄など）
        
    - 署名付きJWT: `{sid, ver, checksum, owner, created_at}`
        
    - 公開URL: `/r/abc?t=<jwt>` → FastAPIで検証・ClickHouseへイベント記録。
    
    - KPI: ①公開率（Run→Publish）②公開→クリックCTR③クリック→フォーク率④フォーク→新規登録率。