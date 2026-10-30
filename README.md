# App Store Review Skill for Claude Code

A [Claude Code](https://claude.com/claude-code) skill that audits your iOS / iPadOS / macOS app project against
[Apple's App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
**before** you submit, so you catch rejection reasons early.

It scans your code and produces a report like this:

| Status | Guideline | Finding | Evidence | Fix |
|---|---|---|---|---|
| ❌ FAIL | 5.1.1(v) | No in-app account deletion | `AuthService.swift:42` | Add Settings → Delete Account |
| ⚠️ WARN | 2.5.2 | OTA updates enabled | `app.json: updates.enabled` | Use only for bug fixes |
| ✅ PASS | 4.8 | Sign in with Apple offered alongside Google | `LoginView.swift:18` | — |

## What it checks

All five sections of the guidelines:

- **1. Safety** – user-generated / AI content moderation (filter, report, block), medical disclaimers, hard-coded secrets, insecure HTTP
- **2. Performance** – placeholders and test data, app name length, private APIs, remote code / OTA updates, background modes, ads
- **3. Business** – In-App Purchase usage, Restore Purchases, subscription paywall requirements, review prompts
- **4. Design** – copycats, minimum functionality (WebView-only apps), push notifications, Sign in with Apple
- **5. Legal** – privacy policy, `NS…UsageDescription` purpose strings, in-app account deletion, data sent to third-party AI, App Tracking Transparency, `PrivacyInfo.xcprivacy`, kids, location, IP

Plus a **manual checklist** for things that can't be verified from code (screenshots, App Privacy answers, demo account, review notes…).

Supported project types: native Swift/Objective-C, React Native / Expo, Flutter, Capacitor / Cordova.

The skill reads the live guidelines page first (when web access is available) and reports any rule that changed.

## Installation

### Option 1 – one command (macOS / Linux)

```bash
mkdir -p ~/.claude/skills/app-store-review && \
curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/app-store-review-skill/main/skills/app-store-review/SKILL.md \
  -o ~/.claude/skills/app-store-review/SKILL.md
```

### Option 2 – clone

```bash
git clone https://github.com/YOUR_USERNAME/app-store-review-skill.git
mkdir -p ~/.claude/skills
cp -r app-store-review-skill/skills/app-store-review ~/.claude/skills/
```

### Windows (PowerShell)

```powershell
mkdir "$env:USERPROFILE\.claude\skills\app-store-review" -Force
Invoke-WebRequest https://raw.githubusercontent.com/YOUR_USERNAME/app-store-review-skill/main/skills/app-store-review/SKILL.md `
  -OutFile "$env:USERPROFILE\.claude\skills\app-store-review\SKILL.md"
```

To use it in a single project only, put the folder in `<project>/.claude/skills/app-store-review/` instead.

## Usage

Open your app's project folder, start Claude Code and run:

```
/app-store-review
```

or just ask: *"Check this app against the App Store guidelines before I submit."*

Restart any running Claude Code session after installing so the skill is picked up.

## Disclaimer

This is an unofficial tool and is not affiliated with or endorsed by Apple. It reduces the chance of rejection but cannot guarantee approval — App Review makes the final decision. Always read the current guidelines.

---

## Türkçe

Uygulamanı App Store'a göndermeden önce Apple'ın inceleme kurallarına göre tarayan bir Claude Code skill'i. Güvenlik, performans, iş modeli, tasarım ve hukuk bölümlerinin hepsini kontrol eder; her bulguyu guideline numarası, dosya:satır kanıtı ve düzeltme önerisiyle raporlar.

**Kurulum (Mac):**

```bash
mkdir -p ~/.claude/skills/app-store-review && \
curl -fsSL https://raw.githubusercontent.com/YOUR_USERNAME/app-store-review-skill/main/skills/app-store-review/SKILL.md \
  -o ~/.claude/skills/app-store-review/SKILL.md
```

**Kullanım:** Proje klasöründe Claude Code'u aç ve `/app-store-review` yaz.

Resmi bir Apple aracı değildir; onay garantisi vermez.

## License

MIT
