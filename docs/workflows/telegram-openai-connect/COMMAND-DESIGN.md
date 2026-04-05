# Command Design — Telegram OpenAI Connect Helper

Dokumen ini menjelaskan desain command helper `/openai` agar tetap nyaman dipakai, aman terhadap command bawaan, dan mudah dikembangkan.

---

## Tujuan Desain
Command helper ini dibuat untuk mengurangi friction saat reconnect OpenAI, terutama ketika user tidak ingin harus masuk ke VPS hanya untuk memulai auth ulang.

Target desain:
- simple untuk user,
- additive terhadap native commands,
- aman secara state,
- jelas secara error handling,
- bisa dinaikkan dari assistant-driven ke parser yang lebih native.

---

## Prinsip Desain
- **Pelengkap**, bukan pengganti native commands
- **Additive only**
- **Low-friction UX**
- **Safe by default**
- **State-aware**
- **Minimal exposure of secrets**

---

## Command Surface
### Entry command utama
```text
/openai
```
Default behavior:
- jika belum ada subcommand, sistem bisa menampilkan helper atau langsung memulai flow `connect`
- untuk UX paling sederhana, `/openai` bisa dianggap alias ke `/openai connect`

### Subcommand yang disarankan
```text
/openai connect
/openai paste <redirect_url>
/openai status
/openai use <model>
/openai cancel
```

---

## Desain Per Command

## 1. `/openai connect`
### Tujuan
Memulai OAuth flow.

### Behavior
- cek apakah sudah ada auth aktif
- kalau belum, generate OAuth URL
- simpan state `oauth_started` → `awaiting_callback_url`
- kirim instruksi singkat ke user
- kalau callback browser localhost gagal (muter/no redirect), arahkan ke device auth fallback

### Fallback behavior (remote/headless)
- Jalankan `codex login --device-auth`
- Kirim user ke `https://auth.openai.com/codex/device`
- Kirim one-time device code
- Tunggu sukses login dari terminal
- lanjutkan sinkronisasi profile auth di OpenClaw

### Output yang diharapkan
- user mendapat link login
- user tahu langkah berikutnya: kirim callback URL

### Guardrail
- jangan kirim token mentah
- jangan simpan state tanpa expiry

---

## 2. `/openai paste <redirect_url>`
### Tujuan
Menerima callback URL hasil login dari browser.

### Behavior
- verifikasi bahwa session sedang `awaiting_callback_url`
- validasi format redirect URL
- mask parameter sensitif di log
- exchange auth code/token
- jika sukses → `auth_completed`
- jika gagal → `auth_failed`

### Output yang diharapkan
- sukses: auth aktif
- gagal: user diarahkan retry

### Guardrail
- jangan proses callback jika state tidak valid
- jangan echo code/token ke chat
- jangan menerima URL dari domain yang tidak sesuai

---

## 3. `/openai status`
### Tujuan
Menampilkan status auth saat ini.

### Behavior
- cek apakah provider auth aktif
- tampilkan status singkat
- jika ada pending flow, tampilkan bahwa user sedang menunggu callback / flow expired / dsb.

### Output yang diharapkan
Contoh:
- connected
- not connected
- awaiting callback
- auth failed

---

## 4. `/openai use <model>`
### Tujuan
Memilih model OpenAI yang akan dipakai setelah auth aktif.

### Behavior
- hanya bisa dipakai jika auth sudah aktif
- validasi nama model
- set model yang dipilih
- tampilkan konfirmasi berhasil / gagal

### Guardrail
- jika auth belum aktif, arahkan ke `/openai connect`
- validasi model sebelum disimpan

---

## 5. `/openai cancel`
### Tujuan
Membatalkan flow yang sedang berjalan.

### Behavior
- jika ada pending state, ubah ke `cancelled`
- bersihkan state sementara
- kirim pesan singkat bahwa flow dibatalkan

### Alasan penting
Ini mencegah state menggantung terlalu lama.

---

## State Model
State minimum yang disarankan:
- `idle`
- `oauth_started`
- `awaiting_callback_url`
- `callback_received`
- `auth_completed`
- `auth_failed`
- `cancelled`
- `expired`

### Aturan utama
- setiap state harus punya transisi yang jelas
- setiap transisi harus punya trigger yang jelas
- state harus punya expiry time

---

## UX Copy Guidance
Command helper harus memakai copy yang singkat dan jelas.

### Good UX principles
- langsung bilang apa yang harus dilakukan user
- jangan terlalu verbose
- kalau gagal, beri next step yang jelas
- kalau sukses, beri konfirmasi dan opsi berikutnya

### Contoh sukses connect
> Auth link sudah siap. Silakan login lewat browser, lalu kirim callback URL ke sini.

### Contoh gagal callback
> Callback URL tidak valid atau sudah expired. Jalankan `/openai connect` untuk memulai ulang.

### Contoh status
> OpenAI auth aktif. Anda bisa lanjut memilih model dengan `/openai use <model>`.

---

## Error Handling Design
Command harus menangani error berikut:
- missing args,
- malformed callback,
- expired state,
- provider exchange failure,
- duplicate connect while flow still pending,
- model invalid,
- user cancel mid-flow.
- device code authorization belum aktif di akun ChatGPT.

### Known blocker: Device code authorization belum aktif
Gejala:
- user klik Continue di browser flow, tapi redirect callback localhost tidak jalan.

Penanganan:
1. Minta user klik link merah: `Enable device code authorization for Codex`.
2. User diarahkan ke ChatGPT Security Settings.
3. User aktifkan opsi: `Enable device code authorization for Codex`.
4. Jalankan ulang `codex login --device-auth`.
5. Ulangi verifikasi di halaman device code.

### Prinsip error message
- singkat,
- tidak bocorkan detail sensitif,
- selalu beri next step.

---

## Security Design
- callback URL divalidasi sebelum diproses
- token/code tidak pernah di-echo ke chat
- parameter sensitif dimask di log
- state pending tidak disimpan terlalu lama
- helper command tidak boleh membuka akses melewati scope auth yang sudah ada

---

## Compatibility Design
Agar aman dengan native Telegram commands:
- helper command ditambahkan via `customCommands`
- tidak menyentuh `commands.native`
- tidak menyentuh `commands.nativeSkills`
- tidak mengganti seluruh command set yang sudah ada

---

## Recommended Roadmap
### Fase 1
- `/openai`
- `/openai connect`
- `/openai paste <url>`
- `/openai status`
- state machine dasar
- expiry logic

### Fase 2
- `/openai use <model>`
- lebih baik error handling
- lebih rapi logging & masking

### Fase 3
- native parser arg yang lebih matang
- persistent session state
- command UX polish

---

## Kesimpulan
Desain command ini harus memprioritaskan tiga hal:
1. **praktis buat user**,
2. **aman buat command bawaan**,
3. **stabil secara state dan error handling**.

Kalau tiga hal ini dijaga, helper `/openai` bisa jadi command operasional yang benar-benar berguna tanpa bikin integrasi Telegram jadi rapuh.
