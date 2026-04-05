# Rama Claw

Kumpulan petunjuk, workflow, checklist, dan catatan desain untuk kebutuhan operasional digital.

Repo ini ditujukan sebagai tempat dokumentasi publik yang rapi, praktis, dan bisa langsung dipakai sebagai referensi kerja.

## Struktur repo
- `docs/README.md` → peta dokumentasi
- `docs/workflows/` → kumpulan workflow
- setiap workflow disimpan dalam folder terpisah agar semua konteksnya utuh di satu tempat

## Pola dokumentasi
Setiap workflow idealnya punya struktur seperti ini:

```text
docs/workflows/<workflow-name>/
  README.md
  CHECKLIST.md
  COMMAND-DESIGN.md
  NOTES.md
```

## Isi saat ini
- workflow integrasi OAuth via Telegram
- alasan kenapa helper flow ini dibuat
- checklist implementasi agar helper command tetap aman
- command design untuk `/openai` dan subcommand turunannya

## Prinsip dokumentasi
- fokus ke langkah yang bisa dipakai ulang
- hindari detail internal yang sensitif
- jaga agar contoh implementasi tetap aman untuk dipublikasikan
- prioritaskan dokumen yang praktis, bukan teori berlebihan
- workflow-centric: satu workflow, satu folder, semua konteksnya ada di dalam

## Rencana isi berikutnya
- best practice operasional
- notes troubleshooting
- checklist deployment
- pattern command helper lain
