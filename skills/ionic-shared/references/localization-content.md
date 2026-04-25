# Translation Content (TR / EN)

Drop these into the framework-appropriate location:

| Framework | Path |
|-----------|------|
| Angular   | `src/assets/i18n/{en,tr}.json` |
| React     | `src/i18n/{en,tr}.json`        |
| Vue       | `src/assets/i18n/{en,tr}.json` |

The keys here cover every string used by the onboarding, paywall, tabs, and settings pages in all three framework skills.

## `en.json`

```json
{
  "tabs": {
    "home": "Home",
    "explore": "Explore",
    "settings": "Settings"
  },
  "onboarding": {
    "next": "Next",
    "start": "Get Started",
    "skip": "Skip"
  },
  "paywall": {
    "title": "Go Premium",
    "weekly": "Weekly",
    "yearly": "Yearly",
    "subscribe": "Subscribe",
    "skip": "Continue with ads",
    "restore": "Restore Purchases",
    "loadError": "Couldn't load subscription options.",
    "retry": "Retry",
    "noOfferings": "No subscriptions are available right now.",
    "purchaseError": "Purchase failed. Please try again.",
    "restoreEmpty": "No previous purchases found.",
    "restoreError": "Couldn't restore purchases. Please try again."
  },
  "settings": {
    "title": "Settings",
    "language": "Language",
    "theme": "Theme",
    "system": "System",
    "light": "Light",
    "dark": "Dark",
    "notifications": "Notifications",
    "removeAds": "Remove Ads",
    "resetOnboarding": "Reset Onboarding"
  }
}
```

## `tr.json`

```json
{
  "tabs": {
    "home": "Ana Sayfa",
    "explore": "Keşfet",
    "settings": "Ayarlar"
  },
  "onboarding": {
    "next": "İleri",
    "start": "Başla",
    "skip": "Atla"
  },
  "paywall": {
    "title": "Premium'a Geç",
    "weekly": "Haftalık",
    "yearly": "Yıllık",
    "subscribe": "Abone Ol",
    "skip": "Reklamlı devam et",
    "restore": "Satın Alımları Geri Yükle",
    "loadError": "Abonelik seçenekleri yüklenemedi.",
    "retry": "Tekrar dene",
    "noOfferings": "Şu anda kullanılabilir abonelik yok.",
    "purchaseError": "Satın alma başarısız oldu. Lütfen tekrar deneyin.",
    "restoreEmpty": "Önceki satın alma bulunamadı.",
    "restoreError": "Satın alımlar geri yüklenemedi. Lütfen tekrar deneyin."
  },
  "settings": {
    "title": "Ayarlar",
    "language": "Dil",
    "theme": "Tema",
    "system": "Sistem",
    "light": "Açık",
    "dark": "Koyu",
    "notifications": "Bildirimler",
    "removeAds": "Reklamları Kaldır",
    "resetOnboarding": "Tanıtımı Sıfırla"
  }
}
```

## Turkish characters — non-negotiable

When writing Turkish strings you MUST use the correct characters. Do not approximate.

- `ı` (lowercase dotless i) — NOT `i`
- `İ` (uppercase dotted I) — NOT `I`
- `ü Ü ö Ö ç Ç ş Ş ğ Ğ`

| ✅ Correct | ❌ Wrong |
|-----------|---------|
| Ayarlar   | Ayarlar (same word — but check the real diff cases below) |
| Giriş     | Giris   |
| Çıkış     | Cikis   |
| Başla     | Basla   |
| İleri     | Ileri   |
| Güncelle  | Guncelle |
