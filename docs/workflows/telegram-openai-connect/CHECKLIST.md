# Implementation Checklist — Telegram OpenAI Connect Flow

Checklist ini dipakai untuk memastikan implementasi command helper `/openai` di Telegram tetap aman, additive, dan siap dipakai operasional.

---

## 1. Pre-Implementation
- [ ] Pastikan tujuan flow jelas: memudahkan reconnect OpenAI tanpa harus login ke VPS setiap kali.
- [ ] Pastikan helper command bersifat **pelengkap**, bukan pengganti command native.
- [ ] Pastikan dokumentasi guardrail sudah dibaca sebelum menyentuh config.
- [ ] Verifikasi bahwa native Telegram commands saat ini masih tampil normal.
- [ ] Verifikasi environment target memang memakai Telegram command menu native.

---

## 2. Config Safety Checklist
- [ ] **Jangan** set `commands.native=false`
- [ ] **Jangan** set `channels.telegram.commands.native=false`
- [ ] **Jangan** set `commands.nativeSkills=false` kecuali memang diminta eksplisit
- [ ] Gunakan pendekatan **additive only**
- [ ] Tambahkan helper command melalui `channels.telegram.customCommands`
- [ ] Pastikan tidak ada config yang berpotensi wipe `setMyCommands([])`

---

## 3. Command Menu Checklist
- [ ] `/openai` muncul di custom menu Telegram
- [ ] Built-in command bawaan tetap muncul
- [ ] Tidak ada command native yang hilang setelah patch config
- [ ] Description command cukup jelas untuk user

Contoh target helper:
- `/openai connect device-auth`
- `/openai connect url`
- `/openai paste <url>`
- `/openai status`
- `/openai use <model>`
- `/openai cancel`

---

## 4. Flow Checklist
### 4.1 Connect
- [ ] `/openai connect device-auth` memulai device auth flow
- [ ] `/openai connect url` memulai browser callback flow
- [ ] OAuth URL berhasil dibuat dan dikirim ke user
- [ ] User mendapat instruksi yang jelas untuk login via browser
- [ ] Jika browser callback localhost gagal/muter, flow fallback ke device auth tersedia
- [ ] Instruksi fallback mengarah ke `codex login --device-auth` + halaman verifikasi device code

### 4.2 Callback
- [ ] Callback URL valid bisa diterima
- [ ] Callback URL invalid ditolak aman
- [ ] Callback URL expired ditolak dengan instruksi retry
- [ ] Callback tidak memproses token mentah ke chat

### 4.3 Status
- [ ] `/openai status` menampilkan auth state yang benar
- [ ] User bisa tahu apakah auth aktif, gagal, atau perlu ulang

### 4.4 Use Model
- [ ] `/openai use <model>` hanya bisa dipakai setelah auth aktif
- [ ] Model selection error ditangani dengan aman
- [ ] Hasil set model bisa diverifikasi

---

## 5. State Handling Checklist
- [ ] State flow disimpan per user / per session
- [ ] State minimum tersedia: `idle`, `oauth_started`, `awaiting_callback_url`, `auth_completed`, `auth_failed`, `cancelled`, `expired`
- [ ] State dibersihkan saat flow selesai
- [ ] State dibersihkan saat user cancel
- [ ] State dibersihkan saat timeout / expired

---

## 6. Timeout / Expiry Checklist
- [ ] Pending OAuth punya expiry time yang jelas (disarankan 10–15 menit)
- [ ] Callback setelah expiry tidak diproses sebagai auth valid
- [ ] User diarahkan untuk menjalankan flow ulang

---

## 7. Error Handling Checklist
- [ ] User kirim callback tanpa connect → ditolak aman
- [ ] User kirim callback rusak / salah → ditolak aman
- [ ] OAuth provider gagal → flow masuk `auth_failed`
- [ ] User batalkan flow → state masuk `cancelled`
- [ ] Restart di tengah flow → state tetap konsisten atau invalidasi aman
- [ ] Kasus `Enable device code authorization for Codex` belum aktif ditangani dengan instruksi yang benar
- [ ] User diarahkan ke ChatGPT Security Settings untuk mengaktifkan device code authorization
- [ ] Setelah user enable, flow meminta rerun `codex login --device-auth`

---

## 8. Security Checklist
- [ ] Token mentah tidak pernah dikirim ke chat
- [ ] Parameter sensitif di callback URL dimask di log
- [ ] Jangan simpan data yang tidak perlu
- [ ] Log sensitif tidak dibocorkan ke user
- [ ] Validasi callback URL dilakukan sebelum exchange

---

## 9. Testing Checklist
### Functional
- [ ] `/openai` memulai flow
- [ ] `/openai status` jalan
- [ ] `/openai use <model>` jalan setelah auth aktif
- [ ] Callback valid sukses diproses
- [ ] Callback invalid gagal aman
- [ ] Device auth sukses saat browser OAuth callback tidak usable di remote/headless

### Regression
- [ ] Native command Telegram tetap ada
- [ ] Helper command tidak menghapus menu existing
- [ ] Restart gateway tidak merusak command menu
- [ ] Restart gateway tidak bikin state ambigu

### UX
- [ ] Pesan instruksi cukup jelas
- [ ] Pesan error tidak membingungkan
- [ ] Pesan sukses memberi next action yang jelas

---

## 10. Rollback Checklist
- [ ] Hapus entri `channels.telegram.customCommands` untuk `openai`
- [ ] Restart gateway secara terkontrol
- [ ] Verifikasi native command bawaan tetap ada
- [ ] Verifikasi `/openai` tidak lagi muncul di menu custom
- [ ] Verifikasi tidak ada side effect menu kosong

---

## 11. Done Criteria
Implementasi dianggap siap jika:
- [ ] helper `/openai` aktif
- [ ] native commands aman
- [ ] connect flow bisa dipakai
- [ ] callback flow aman
- [ ] status auth bisa dicek
- [ ] state machine bekerja stabil
- [ ] timeout & rollback teruji
- [ ] tidak ada kebocoran data sensitif di chat atau log
