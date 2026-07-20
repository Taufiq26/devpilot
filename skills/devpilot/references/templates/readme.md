<!-- Template: this file must stay tool/skill-agnostic — any AI agent or human
     should be able to follow it without knowing devpilot exists. Fill every
     {placeholder} from config.md and the other core docs. List ONLY the docs
     marked active in config.md § Active documents — skip inactive ones
     entirely, don't list them as "N/A" or explain why they're missing (that
     belongs in config.md, not here). Strip this instructional comment block
     and every other HTML comment from the generated output. This template is
     never skipped — unlike core-*.md docs it does not depend on project type. -->
# {Project Name} — Panduan Dokumentasi

Dokumen ini untuk **AI Agent atau siapa pun** yang mengerjakan implementasi
{Project Name} — tidak terikat tool/skill tertentu. Baca ini dulu sebelum
menulis kode apa pun.

## Struktur

- **`{docs}/core/`** — sumber kebenaran utama (source of truth). Semua
  keputusan implementasi merujuk ke sini, bukan ke ingatan/asumsi.
  - `config.md` — konfigurasi proyek + log keputusan — baca ini duluan.
  <!-- Repeat one bullet per doc marked active in config.md § Active
       documents, in that table's order, each with a one-sentence purpose
       pulled from the doc's own opening section. -->
  - `{doc-name}.md` — {one-sentence purpose}.
  - `phases.md` — **execution plan**: fase & task bernomor, dependency-aware.
  - `backlog.md` — item yang ditunda beserta analisis dampaknya.
- **`{docs}/formal/`** — dokumen presentasi (PRD/SRS/gantt) diturunkan dari
  `core/` untuk stakeholder/manusia — bukan working source, jangan diedit
  langsung (edit `core/` lalu regenerate lewat `/devpilot docs`).

## Cara kerja yang diharapkan

1. Baca `core/config.md` (decision log) dan `core/requirements.md` dulu untuk
   memahami konteks & keputusan yang sudah diambil.
2. Kerjakan `core/phases.md` **berurutan**. Jangan lompat fase kecuali
   dependency-nya sudah terpenuhi.
3. Centang (`- [x]`) task di `phases.md` **segera** setelah selesai — jangan
   menumpuk update di akhir fase.
4. Buat 1 commit git per fase yang selesai, referensikan nomor fase di pesan
   commit.
5. <!-- Include this bullet ONLY if phases.md has any task tagged as external
        coordination / owned by another team or repo; otherwise omit it
        entirely rather than leaving a placeholder. -->
   Task yang ditandai **[{external-coordination tag used in phases.md}]**
   bukan pekerjaan repo ini — itu dependency yang harus dikoordinasikan
   dengan tim/repo terkait, jangan diimplementasikan di sini.
6. Kalau menemukan kebutuhan baru atau ambiguitas yang tidak terjawab dokumen
   ini: jangan berasumsi diam-diam. Catat ke `backlog.md` (jika ditunda) atau
   tanyakan ke pemilik proyek, lalu **update dokumen `core/` yang relevan** —
   dokumen ini harus tetap hidup & akurat, bukan snapshot yang boleh basi.

## Catatan penting

<!-- 2-5 bullets max: pull from config.md's decision log (or requirements.md)
     the items most likely to be misread or silently assumed away by someone
     skimming the docs cold — naming conventions that look like literal paths
     but aren't, hard "must"/"must not" boundaries, non-obvious scope limits,
     things an external system is or isn't allowed to touch. This is a
     highlight reel, not a copy of the full decision log — if everything in
     config.md seems equally important, pick the ones a reasonable engineer
     would get wrong on first read, not the full list. -->
- {…}
