# OpenAI Connect Flow di Telegram (Aman untuk Command Bawaan)

## Tujuan
Dokumen ini merangkum cara menambahkan flow OpenAI OAuth lewat Telegram tanpa mengganggu command bawaan yang sudah ada.

Target utamanya:
- command bawaan tetap aman,
- penambahan command dilakukan secara additive,
- flow bisa dikembangkan bertahap dari assistant-driven ke handler yang lebih native.

---

## Prinsip Utama
Agar command bawaan tetap aman, implementasi harus mengikuti prinsip:
- **additive only**
- **no override**
- **no command wipe**

Artinya:
- jangan mematikan native commands,
- jangan mengganti seluruh menu command,
- jangan memakai pendekatan yang mengosongkan command existing.

---

## Ringkasan Flow
Flow operasional sederhananya seperti ini:
1. User mengetik `/openai`
2. Sistem memulai flow login OAuth
3. User membuka URL login di browser
4. User mengirim callback URL kembali ke chat
5. Sistem memproses callback dan menyelesaikan auth
6. User bisa cek status auth atau memilih model yang akan dipakai

---

## Contoh Command Tambahan
Command custom yang ditambahkan bisa berupa:

```json
{
  "channels": {
    "telegram": {
      "customCommands": [
        {
          "command": "openai",
          "description": "OpenAI OAuth helper: connect|paste|status|use"
        }
      ]
    }
  }
}
```

### Catatan
- command ini hanya menambah menu `/openai`,
- command bawaan tetap dibiarkan aktif,
- konfigurasi dilakukan secara additive.

---

## Current Behavior vs Target Behavior

### Current behavior
- `/openai` sudah bisa dipakai untuk memulai flow
- callback masih bisa diproses lewat assistant-driven flow di chat
- belum semua subcommand dipisah menjadi parser native tersendiri

### Target behavior
- `/openai connect`
- `/openai paste <url>`
- `/openai status`
- `/openai use <model>`

Dengan pola ini, flow akan lebih rapi, lebih predictable, dan lebih gampang dites.

---

## State Machine yang Disarankan
Agar implementasi stabil, sebaiknya flow punya state yang jelas per user atau per session.

### State
- `idle`
- `oauth_started`
- `awaiting_callback_url`
- `callback_received`
- `auth_completed`
- `auth_failed`
- `cancelled`
- `expired`

### Kenapa penting
- callback tidak nyasar ke flow yang salah,
- restart atau timeout lebih gampang ditangani,
- debugging jadi lebih sederhana.

---

## Timeout / Expiry Rule
Disarankan:
- pending OAuth hanya valid **10–15 menit**,
- kalau user kirim callback setelah expired, flow harus diulang,
- state expired atau cancelled harus dibersihkan aman.

---

## Error Cases yang Wajib Ditangani
Minimal case berikut perlu diproses dengan aman:
- user kirim callback tanpa memulai connect,
- callback URL invalid,
- callback expired,
- login dibatalkan user,
- auth provider gagal,
- user sudah punya auth aktif sebelumnya,
- restart terjadi di tengah flow.

---

## Success Criteria
Flow dianggap berhasil jika:
- OAuth exchange sukses,
- credential tersimpan,
- status auth bisa diverifikasi,
- user mendapat konfirmasi bahwa auth sudah aktif,
- optional: model pilihan berhasil diset.

---

## Post-Auth Flow
Setelah auth selesai, urutan yang disarankan:
1. verifikasi auth berhasil,
2. cek status auth,
3. optional pilih model,
4. kirim konfirmasi sukses,
5. bersihkan state sementara.

---

## Test Checklist
Checklist minimum:
- `/openai` muncul di custom menu
- native commands bawaan tetap muncul
- flow login berhasil dimulai
- callback valid diterima
- callback invalid ditolak aman
- callback expired ditolak dengan instruksi retry
- cancel flow tidak merusak state
- restart tidak menghapus native commands
- status auth bisa dicek
- flow selesai tidak meninggalkan pending state

---

## Rollback
Kalau ingin menghapus command custom:
1. hapus entri `customCommands` untuk `openai`,
2. restart secara terkontrol,
3. verifikasi native command tetap ada,
4. pastikan tidak ada side effect pada command menu.

---

## Catatan Keamanan
- jangan tampilkan token mentah ke chat,
- mask parameter sensitif di log,
- callback URL boleh diproses, tapi detail sensitifnya jangan dipublikasikan,
- jangan menyimpan data yang tidak perlu,
- gunakan validasi input yang ketat untuk callback URL.

---

## Kesimpulan
Flow ini cocok dipakai kalau ingin menambahkan integrasi OAuth lewat Telegram **tanpa mengorbankan command bawaan**. Kuncinya ada di:
- additive configuration,
- state yang jelas,
- expiry rule,
- error handling,
- dan test checklist yang disiplin.
