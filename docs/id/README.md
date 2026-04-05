# Peta Dokumentasi (Indonesia)

Dokumentasi publik dengan fokus pada workflow, checklist implementasi, desain command, dan catatan konteks operasional.

## Struktur Utama

### `docs/id/workflows/`
Dokumentasi disusun **per workflow**, bukan dipisah per jenis dokumen.

Tujuannya:
- satu workflow punya semua konteks di satu tempat,
- pembaca tidak perlu loncat folder,
- lebih scalable saat jumlah workflow bertambah.

## Pola Folder Workflow

```text
docs/id/workflows/<workflow-name>/
  README.md
  CHECKLIST.md
  COMMAND-DESIGN.md
  NOTES.md
```

### Penjelasan Isi
- `README.md` → alur utama workflow
- `CHECKLIST.md` → checklist implementasi dan validasi
- `COMMAND-DESIGN.md` → desain command / perilaku interaksi
- `NOTES.md` → alasan, konteks, dan catatan tambahan

## Workflow yang Sudah Ada
### `docs/id/workflows/telegram-openai-connect/`
- `README.md`
- `CHECKLIST.md`
- `COMMAND-DESIGN.md`
- `NOTES.md`

## Cara Pakai
1. buka folder workflow yang relevan,
2. baca `README.md` untuk paham alur utama,
3. pakai `CHECKLIST.md` untuk implementasi,
4. lihat `COMMAND-DESIGN.md` untuk keputusan desain,
5. baca `NOTES.md` untuk konteks tambahan.

## Catatan Kompatibilitas
Path lama `docs/workflows/` tetap dipertahankan agar link lama tidak putus.
