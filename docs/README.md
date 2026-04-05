# Documentation Map

Repo ini menyimpan dokumentasi publik yang fokus ke workflow, checklist, desain implementasi, dan catatan konteks.

## Struktur utama

### `docs/workflows/`
Semua dokumentasi disusun **per workflow**, bukan dipisah per jenis dokumen.

Tujuannya:
- satu workflow punya semua konteks di satu tempat,
- pembaca tidak perlu loncat folder,
- lebih scalable saat jumlah workflow bertambah.

## Pola folder workflow
Setiap workflow disarankan mengikuti pola:

```text
docs/workflows/<workflow-name>/
  README.md
  CHECKLIST.md
  COMMAND-DESIGN.md
  NOTES.md
```

### Penjelasan isi
- `README.md` → alur utama workflow
- `CHECKLIST.md` → checklist implementasi dan validasi
- `COMMAND-DESIGN.md` → desain command / perilaku interaksi
- `NOTES.md` → alasan, konteks, dan catatan tambahan

## Workflow yang sudah ada
### `docs/workflows/telegram-openai-connect/`
- `README.md`
- `CHECKLIST.md`
- `COMMAND-DESIGN.md`
- `NOTES.md`

## Cara pakai
1. buka folder workflow yang relevan,
2. baca `README.md` untuk paham alur utama,
3. pakai `CHECKLIST.md` untuk implementasi,
4. lihat `COMMAND-DESIGN.md` untuk keputusan desain,
5. baca `NOTES.md` untuk konteks tambahan.
