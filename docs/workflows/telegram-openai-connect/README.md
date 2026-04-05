# OpenAI Connect Flow di Telegram (Aman untuk Command Bawaan)

## Tujuan
Dokumen ini merangkum cara menambahkan flow OpenAI OAuth lewat Telegram tanpa mengganggu command bawaan yang sudah ada.

Target utamanya:
- command bawaan tetap aman,
- penambahan command dilakukan secara additive,
- flow `/openai` berjalan deterministic (native command handler), bukan reasoning agent.

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
1. User mengetik `/openai connect device-auth` atau `/openai connect url`
2. Sistem memulai flow auth sesuai mode
3. User menyelesaikan verifikasi
4. User cek `/openai status`
5. User bisa pilih model dengan `/openai use <model>`

Flow fallback untuk remote/headless:
1. Jika browser callback localhost muter terus atau gagal redirect, jangan dipaksa.
2. Pindah ke device auth: jalankan `codex login --device-auth`.
3. User buka `https://auth.openai.com/codex/device` dan masukkan one-time code.
4. Setelah sukses, OpenClaw bisa sync profile OAuth dari Codex CLI.

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
          "description": "OpenAI helper: connect device-auth | connect url | status | cancel"
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

## Current Behavior (Live)

Per hari ini, behavior live yang sudah diterapkan:
- `/openai connect device-auth`
- `/openai connect url`
- `/openai paste <url>` (khusus mode `connect url`)
- `/openai status`
- `/openai use <model>`
- `/openai cancel`

Semua command di atas ditangani deterministic via native command handler plugin (bypass reasoning agent).

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
- **device code authorization belum di-enable** di ChatGPT Security Settings.

### Blocker yang sering muncul di lapangan
Kasus real yang harus didukung:
- user klik Continue di browser flow, tapi halaman hanya muter dan tidak redirect callback.
- ini umum di remote/VPS/headless atau network policy yang memblok localhost callback.

Fix yang dipakai:
1. Buka link merah pada halaman OpenAI: `Enable device code authorization for Codex`.
2. User akan diarahkan ke ChatGPT Security Settings.
3. Di bagian paling bawah, aktifkan opsi:
   `Enable device code authorization for Codex`.
4. Jalankan ulang command:
   `codex login --device-auth`
5. Selesaikan login via `https://auth.openai.com/codex/device` + one-time code.

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
- flow login mode `connect device-auth` berhasil dimulai
- flow login mode `connect url` berhasil dimulai
- callback valid diterima
- callback invalid ditolak aman
- callback expired ditolak dengan instruksi retry
- device auth path berhasil saat browser callback gagal
- case "device code authorization belum enabled" punya instruksi fix yang jelas
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
