# Draft: GitHub Action build mame2003-plus PS3 SELF

Workflow: `build-mame2003-plus-ps3.yml` — **DRAFT, belum pernah di-run.**

## Cara pakai
1. Taruh file workflow di `.github/workflows/` repo milik lo
   (misal fork `libretro/mame2003-plus-libretro`, atau repo build khusus).
2. GitHub → Actions → "Build mame2003-plus PS3 SELF" → Run workflow.
   Default: core + frontend RetroArch **master terbaru** (bisa override ref di input).
3. Download artifact `mame2003_plus_libretro_ps3.SELF.zip`, extract `.SELF`-nya,
   copy ke folder `cores` di PS3 (sejajar SELF core CE yang lain).
   CE spawn tiap core sebagai proses standalone via exitspawn — hasil link
   PSL1GHT (`make_self`, CEX) mestinya jalan di CFW/HEN.

## Alur build (di dalam container `reallibretroretroarch/libretro-build-psl1ght`)
1. **Probe toolchain** — set `PS3DEV`/`PSL1GHT`/`PORTLIBS`/`COMMONLV` kalau container
   belum set (default layout ps3dev resmi). Cek log step ini di run pertama.
2. **Clone** mame2003-plus + RetroArch (depth 1).
3. **Core**: `make -f Makefile platform=psl1ght` → `mame2003_plus_libretro_psl1ght.a`
4. **Frontend**: copy `.a` jadi `libretro_psl1ght.a`, `make -f Makefile.psl1ght`
   → `retroarch_psl1ght.self` (di-sign `make_self` oleh ppu_rules).
5. **Package**: rename jadi `mame2003_plus_libretro_ps3.SELF` (+ `BUILD_INFO.txt`
   berisi commit core/frontend), zip, upload artifact.

## Yang sudah diverifikasi dari source (2026-10-06)
- mame2003-plus `Makefile` punya target `platform=psl1ght` (pure C, `CXX` tidak dipakai).
- RetroArch master masih punya `Makefile.psl1ght` + `griffin/griffin.c`;
  `Makefile.ps3` (Sony SDK) sudah hilang dari master.
- psl1ght `ppu_rules`: `%.self: %.elf` pakai `make_self` (CEX) + `fself` (fake).
- Frontend PSL1GHT default content dir `SSNE10001`, core dir `<port>/cores` —
  konsisten dengan layout CE.
- Container image ada di Docker Hub, terakhir update 2026-04-24.

## Yang belum bisa diverifikasi (cek di run pertama)
- Isi env container (apakah `PS3DEV`/`PSL1GHT` sudah di-set, dan apakah toolchain
  menyediakan `ppu-gcc` atau `ppu-lv2-gcc`) → ditangani probe step, tapi baca lognya.
- Template CI resmi libretro (`ci-templates/psl1ght-static.yml`) tidak bisa diakses
  (gitlab.com ke-block Cloudflare challenge dari sini) — workflow ini rekonstruksi
  dari Makefile + ppu_rules, bukan salinan template.
- Build MAME full di runner GitHub (~ribuan file): timeout di-set 180 menit;
  kalau keok, naikkan `timeout-minutes` atau pakai self-hosted runner.
- Hasil SELF **belum dites di PS3 asli** — CE pakai frontend Sony SDK 1.9.1-era,
  sedangkan SELF ini link frontend RetroArch master. Secara arsitektur mestinya
  aman (proses terpisah, tidak ada ABI boundary), tapi butuh test boot di console.

## Bukan bagian workflow ini
- Sony SDK tidak bisa masuk CI publik (proprietary) — makanya jalur PSL1GHT.
- Signing NPDRM / PKG tidak dibuat — cuma SELF mentah buat folder `cores`.
