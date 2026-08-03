# Skills

AI coding agent'lar ile kullanmak icin derlenmis skill koleksiyonu. Claude Code, Cursor, Windsurf ve diger LLM tabanli araclarla uyumludur.

## Kurulum

Bu repo tek kaynak olarak tutulur. Claude Code ve OpenCode ayni `SKILL.md` dosyalarini okuyabilir; skill'leri iki tarafa ayri ayri kopyalayip farkli versiyonlar yaratmak yerine symlink kullanmak daha guvenlidir.

### Claude Code (Global)

```bash
mkdir -p ~/.claude
test ! -e ~/.claude/skills || mv ~/.claude/skills ~/.claude/skills.backup
ln -sfn "$(pwd)" ~/.claude/skills
```

Mevcut skill'leri saklamak istemiyorsaniz backup klasorunu daha sonra silebilirsiniz.

### Claude Code (Proje)

```bash
mkdir -p .claude/skills
cp -r */ .claude/skills/
```

### OpenCode (Global)

Global OpenCode config'ine bu repo path'ini ekleyin:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": ["/Users/alperalin/Desktop/Dev/Projects/skills"]
  }
}
```

Bu dosya global kurulumda `~/.config/opencode/opencode.jsonc` konumunda durur. OpenCode config'i baslangicta yukler; config veya skill dosyalarini degistirdikten sonra OpenCode'u yeniden baslatin.

Not: OpenCode `~/.claude/skills` altindaki skill'leri de otomatik tarar. Claude Code global kurulumundaki symlink bu yuzden OpenCode icin de yeterli olabilir; yine de explicit `skills.paths` ayari hangi repo'nun tek kaynak oldugunu daha net yapar.

### Diger Araclar

Skill dosyalarini (SKILL.md) aracin rules dosyasina veya system prompt'una ekleyin.

## Skill Listesi

### Planlama & Tasarim

| Skill | Aciklama |
|---|---|
| [to-spec](to-spec/) | Mevcut konusmayi interview etmeden spec'e donusturur, issue tracker'a publish eder |
| [to-prd](to-prd/) | Mevcut konusma context'inden interview etmeden PRD uretir, GitHub issue acar |
| [planning-and-task-breakdown](planning-and-task-breakdown/) | Spec'i sizing'li, checkpoint'li task'lara boler |
| [wayfinder](wayfinder/) | Tek oturuma sigmayan buyuk/belirsiz isleri, issue tracker uzerinde karar-bileti haritasi olarak planlar |
| [to-tickets](to-tickets/) | Plan/spec/konusmayi blocking-edge'li tracer-bullet ticket'lara boler, wide-refactor icin expand-contract destegi |
| [prototype](prototype/) | Bir tasarim sorusuna cevap icin throwaway logic (TUI) veya UI (varyant switcher) prototipi kurar |
| [grill-me-docs](grill-me-docs/) | Plan/tasarimi amansizca sorgular, domain modeline karsi challenge eder, CONTEXT.md ve ADR'lari gunceller |

### Gelistirme

| Skill | Aciklama |
|---|---|
| [implement](implement/) | Spec/ticket'i tdd + code-review + commit akisiyla uygular |
| [tdd](tdd/) | Red-green-refactor dongusu ile test-driven development |
| [incremental-implementation](incremental-implementation/) | Ince vertical slice'larla incremental implementasyon |
| [source-driven-development](source-driven-development/) | Her framework kararini resmi dokumantasyona dayandirma |
| [frontend-ui-engineering](frontend-ui-engineering/) | Production-kalite, accessible, AI-aesthetic'ten uzak UI gelistirme |
| [research](research/) | Background agent ile birincil kaynaklara dayali arastirma, bulgulari markdown'a yazar |

### Kalite & Review

| Skill | Aciklama |
|---|---|
| [code-review](code-review/) | 2-eksenli paralel sub-agent review: Standards (repo kurallari + Fowler smell baseline) vs Spec (gereksinim uyumu) |
| [deep-review](deep-review/) | 5-fazli multi-agent audit pipeline: feature catalog, review swarm, critique, arbitration, final report |
| [oop-review](oop-review/) | 5 paralel agent ile OOP/mimari review; false positive'leri onleyen pragmatik filtreler |
| [improve-codebase-architecture](improve-codebase-architecture/) | HTML report ile mimari friction analizi, `/codebase-design` ve `/domain-modeling` ile entegre |
| [diagnosing-bugs](diagnosing-bugs/) | 6-fazli disiplinli debug dongusu: feedback loop, minimize, hipotez, enstrumantasyon, fix, post-mortem |
| [triage](triage/) | Issue/PR'lari triage state machine'i uzerinden yonetir: kategorize et, dogrula, gerekirse grill et, agent-brief yaz |
| [performance-optimization](performance-optimization/) | Olcum-oncelikli performans optimizasyonu ve Core Web Vitals |
| [verification-before-completion](verification-before-completion/) | Kanit olmadan "bitti" demeyi engelleyen dogrulama zorunlulugu |

### Ogrenme

| Skill | Aciklama |
|---|---|
| [teach](teach/) | Stateful ogretim sistemi: mission, lesson, learning record, glossary ve resource pipeline'i ile Zone of Proximal Development tabanli ogrenme |

### Shared (Model-invoked)

| Skill | Aciklama |
|---|---|
| [codebase-design](codebase-design/) | Deep module vocabulary ve prensipleri; `tdd` ve `improve-codebase-architecture` tarafindan kullanilir |
| [domain-modeling](domain-modeling/) | CONTEXT.md ve ADR yonetimi; `grill-me-docs` ve `improve-codebase-architecture` tarafindan kullanilir |
| [grilling](grilling/) | Yeniden kullanilabilir interview loop; `grill-me-docs` ve `improve-codebase-architecture` tarafindan kullanilir |

### Araclar

| Skill | Aciklama |
|---|---|
| [git-guardrails-claude-code](git-guardrails-claude-code/) | Tehlikeli git komutlarini Claude Code hook'lari ile bloklar |
| [extract-skill](extract-skill/) | Konusma context'inden kalici skill dosyasi uretir |
| [handoff](handoff/) | Konusmayi baska bir agent'in devam edebilecegi handoff dokumanina sikistirir |
| [writing-great-skills](writing-great-skills/) | Skill yazma/duzenleme icin referans rehber (invocation, information hierarchy, leading words, failure modes) |

## Kaynaklar

- [Matt Pocock - skills](https://github.com/mattpocock/skills)
- [Addy Osmani - agent-skills](https://github.com/addyosmani/agent-skills)
- [Jesse Vincent - superpowers](https://github.com/obra/superpowers)

## Lisans

MIT
