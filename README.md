# Skills

AI coding agent'lar ile kullanmak icin derlenmis skill koleksiyonu. Claude Code, Cursor, Windsurf ve diger LLM tabanli araclarla uyumludur.

## Kurulum

### Claude Code (Global)

```bash
cp -r */ ~/.claude/skills/
```

### Claude Code (Proje)

```bash
mkdir -p .claude/skills
cp -r */ .claude/skills/
```

### Diger Araclar

Skill dosyalarini (SKILL.md) aracin rules dosyasina veya system prompt'una ekleyin.

## Skill Listesi

### Planlama & Tasarim

| Skill | Aciklama |
|---|---|
| [spec-driven-development](spec-driven-development/) | Kod yazmadan once 4 asamali gated workflow ile spec olusturur |
| [to-prd](to-prd/) | Mevcut konusma context'inden interview etmeden PRD uretir, GitHub issue acar |
| [planning-and-task-breakdown](planning-and-task-breakdown/) | Spec'i sizing'li, checkpoint'li task'lara boler |
| [to-issues](to-issues/) | Plan veya PRD'yi HITL/AFK ayrimli GitHub issue'lara donusturur |
| [grill-me-docs](grill-me-docs/) | Plan/tasarimi amansizca sorgular, domain modeline karsi challenge eder, CONTEXT.md ve ADR'lari gunceller |

### Gelistirme

| Skill | Aciklama |
|---|---|
| [tdd](tdd/) | Red-green-refactor dongusu ile test-driven development |
| [incremental-implementation](incremental-implementation/) | Ince vertical slice'larla incremental implementasyon |
| [source-driven-development](source-driven-development/) | Her framework kararini resmi dokumantasyona dayandirma |
| [frontend-ui-engineering](frontend-ui-engineering/) | Production-kalite, accessible, AI-aesthetic'ten uzak UI gelistirme |

### Kalite & Review

| Skill | Aciklama |
|---|---|
| [review](review/) | 2-eksenli paralel sub-agent review: Standards (repo kurallari) vs Spec (gereksinim uyumu) |
| [deep-review](deep-review/) | 5-fazli multi-agent audit pipeline: feature catalog, review swarm, critique, arbitration, final report |
| [oop-review](oop-review/) | 5 paralel agent ile OOP/mimari review; false positive'leri onleyen pragmatik filtreler |
| [improve-codebase-architecture](improve-codebase-architecture/) | Mimari friction analizi, shallow/deep modul tespiti, deletion test ile refactor onerileri |
| [diagnose](diagnose/) | 6-fazli disiplinli debug dongusu: feedback loop, hipotez, enstrumantasyon, fix, post-mortem |
| [triage-issue](triage-issue/) | Bug'i arastirip TDD-tabanli fix plani ile GitHub issue acar |
| [performance-optimization](performance-optimization/) | Olcum-oncelikli performans optimizasyonu ve Core Web Vitals |
| [verification-before-completion](verification-before-completion/) | Kanit olmadan "bitti" demeyi engelleyen dogrulama zorunlulugu |

### Araclar

| Skill | Aciklama |
|---|---|
| [git-guardrails-claude-code](git-guardrails-claude-code/) | Tehlikeli git komutlarini Claude Code hook'lari ile bloklar |
| [extract-skill](extract-skill/) | Konusma context'inden kalici skill dosyasi uretir |

## Kaynaklar

- [Matt Pocock - skills](https://github.com/mattpocock/skills)
- [Addy Osmani - agent-skills](https://github.com/addyosmani/agent-skills)
- [Jesse Vincent - superpowers](https://github.com/obra/superpowers)

## Lisans

MIT
