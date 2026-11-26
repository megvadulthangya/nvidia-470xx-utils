# Maintainer: Gyöngyösi Gábor <gabor at gshoots dot hu>
# Contributor: Mark Wagie <mark at manjaro dot org>
# Contributor: Philip Müller <philm[at]manjaro[dot]org>
#
# =============================================================================
# NVIDIA 470xx – Debian patch-sorozat mpatch-csel
# For Manjaro-awesome-respin
# =============================================================================
#
#  ┌─────────────────────────────────────────────────────────────────────────┐
#  │  MPATCH – HORDOZHATÓ KÉZIKÖNYV A PKGBUILD-BEN                          │
#  │  (Ha elfelejted, itt minden megtalálható. Nem kell GitHub-ot nyitni.)  │
#  └─────────────────────────────────────────────────────────────────────────┘
#
#  MI ÉS MIÉRT
#  -----------
#  Az mpatch egy fuzzy patchelő. Nem sorszám és nem bájt-pontos kontextus
#  alapján illeszt, hanem hasonlósági algoritmussal keresi a patch helyét.
#  Ez akkor hasznos, ha egy Debian patch egy kicsit elavult a forráshoz
#  képest, DE a változtatás lényege még stimmel.
#
#  KOCKÁZAT
#  --------
#  A fuzzy matching rossz helyre is illeszthet. Ezért:
#    - build-hez MINDIG használj --atomic-ot (minden-vagy-semmi)
#    - a --fuzz-factor-t csak indokolt esetben lazítsd
#    - ha egy patch elhasal: először debug, csak utána lazítás
#
#  ---------------------------------------------------------------------------
#  BEVEZETŐ FORMÁTUMOK (amit az mpatch automatikusan felismer)
#  ---------------------------------------------------------------------------
#   1. Unified Diff          – ez a Debian patchek formátuma (--- a/ +++ b/)
#   2. Markdown diff blokk   – ```diff ... ``` (LLM kimenet)
#   3. Aider Search/Replace  – <<<<<<< SEARCH ... ======= ... >>>>>>> REPLACE
#   4. Conflict Markers      – <<<< ==== >>>> (nincs benne fájlútvonal!)
#
#  A Debian patcheink az 1-es típusba esnek. A 4-es típus CLI-ből
#  csak egy "patch_target" nevű fájlt tud patchelni – nekünk nem jó.
#
#  ---------------------------------------------------------------------------
#  FORMÁTUM-LIMITÁCIÓK (ezeket az mpatch NEM tudja alkalmazni)
#  ---------------------------------------------------------------------------
#  Ha a patch az alábbiak bármelyikét tartalmazza, az mpatch NEM
#  próbálkozik fuzzy-val, hanem azonnal leáll (exit 1, semmit nem ír):
#
#    - Git rename/copy:    "rename from" / "copy from"
#    - Git binary patch:   "GIT binary patch"
#    - Symlink / submodule mód (bármi, ami nem 100644 / 100755)
#    - Context diff:       "diff -c"  → regeneráld "diff -u"-val
#
#  ⚠ EZ NEM FUZZ-PROBLÉMA. Ha egy patch elbukik, ELŐSZÖR ezt kell
#  kizárni, mielőtt a fuzz-factort lazítanád. Ha ezek közül van benne,
#  a --fuzz-factor 0.3 sem segít – a patchet ki kell venni, vagy
#  kézzel át kell írni.
#
#  ---------------------------------------------------------------------------
#  PARANCSONKÉNTI OPCIÓK (mind a PKGBUILD _mpatch_opts tömbben)
#  ---------------------------------------------------------------------------
#
#  --fuzz-factor N            Hasonlósági küszöb, 0.0 – 1.0
#     0.0   = fuzzy KIKAPCSOLVA. Az egzakt és a trailing-whitespace-t
#             toleráló match továbbra is működik. NEM bájt-pontos, és
#             NEM viselkedik úgy, mint a GNU patch. Preview: --dry-run.
#     0.5   = laza (kockázatos, csak debug célra)
#     0.7   = DEFAULT (a tool ajánlott értéke)
#     0.9   = nagyon szigorú (kevés eltűréssel)
#     1.0   = gyakorlatilag csak tökéletes egyezés
#
#     MIKOR MELYIK?
#       - Build-hez: 0.7 (default) vagy 0.9 (ha van idő debugolni)
#       - Ha egy patch elhasal 0.7-en: ELŐSZÖR a FORMÁTUM-LIMITÁCIÓK
#         szekciót kell kizárni (fent), csak utána lazíts 0.5-re.
#         Ha 0.5-ön sikerül, a patch elavult → FRISSÍTENI KELL,
#         nem lazítani.
#       - Ha 0.5 sem elég: a patch kontextusa túlságosan eltér,
#         valószínűleg upstream már beolvasztotta → a patchet ki kell venni
#
#  -a / --atomic / --all-or-nothing   (a kettő UGYANAZ)
#     Minden-vagy-semmi. Ha bármely hunk elhasal, egy fájl sem íródik.
#     Build-hez KÖTELEZŐ. Kikapcsolva részleges írás lehetséges, ami
#     csendben elrontja a forrásfát.
#
#     FONTOS: az atomic és a fuzz-factor ORTOGONÁLIS. Az atomic a
#     write-szemantikát szabályozza, a fuzz a match-szemantikát.
#     Az atomic-ot NEM kell a fuzz-hoz igazítani.
#
#     HATÓKÖR: egy mpatch híváson belül garantál atomitást. A while
#     ciklusban minden patch külön hívás → ha a 40. elhasal, az első
#     39 már lemezre íródott. Ez a mi esetünkben OK, mert a prepare()
#     elején úgyis rm -rf "${_pkg}"-vel tiszta lappal indulunk.
#
#  -R                          Patch megfordítása (add ↔ remove).
#                              Build-hez nem kell, de ha egy patch-et
#                              vissza akarsz fejteni (mit is csinál
#                              pontosan), akkor hasznos.
#
#  --dry-run                   Csak előnézet, nem ír fájlt.
#                              Debug workflow 1. lépése: mit tenne?
#
#  -c / --clipboard            Vágólapról olvas. Ilyenkor az INPUT_FILE
#                              elhagyható: `mpatch -c ./src` is működik.
#                              Build-hez nem kell.
#
#  -                           stdin-ről olvas. Pipe esetén a '-' is
#                              elhagyható: `git diff | mpatch ./src`.
#                              Build-hez nem kell (fájlból olvasunk).
#
#  -v / --verbose              Ismételhető, szintenként:
#                                -v     info (mit csinál épp)
#                                -vv    debug
#                                -vvv   trace
#                                -vvvv  trace + debug-riport
#                              Siker esetén az mpatch ALAPBÓL SEMMIT
#                              nem ír ki. Ha a build logban látni akarod
#                              a patchek előrehaladását, tegyél -v-t a
#                              _mpatch_opts-ba.
#
#  -h / --help                 Súgó.
#
#  ---------------------------------------------------------------------------
#  EXIT KÓDOK ÉS KIMENET (a _mpatch_apply logikája erre épül)
#  ---------------------------------------------------------------------------
#
#    Siker (minden hunk alkalmazva)     → exit 0, semmit nem ír ki
#    Hiba (bármely hunk elbukik)        → exit 1, kiírja a legközelebbi
#                                          találatot + hasonlóságot
#    Nincs patch a bemenetben           → warning + exit 0  ⚠ CSAPDA
#
#  ⚠ A "nincs patch → exit 0" csapda: ha egy Debian patch véletlenül
#  üres, vagy csak kommenteket tartalmaz, az mpatch NÉMÁN átengedi,
#  és a build később, érthetetlen módon hasal el. A _mpatch_apply
#  ezt nem tudja kiszűrni – ha gyanús egy patch, nézd meg a tartalmát.
#
#  ⚠ --atomic NÉLKÜL a sikeres hunok lemezre íródnak akkor is, ha
#  más hunok elbuknak → részleges, csendben elrontott forrásfa.
#  Ezért van --atomic kötelezően a _mpatch_opts-ban.
#
#  ---------------------------------------------------------------------------
#  DEBUG WORKFLOW – HA EGY PATCH ELHASAL
#  ---------------------------------------------------------------------------
#
#  1. LÉPÉS – Nézd meg a hibaüzenetet
#     Az mpatch kiírja, melyik fájlt és melyik hunkot nem tudja alkalmazni.
#     A prepare() automatikusan -vvvv-vel újrafuttatja a hibás patchet,
#     és a debug-riportot a ${srcdir}/mpatch-debug/-be másolja.
#
#  2. LÉPÉS – Kizártad a FORMÁTUM-LIMITÁCIÓT?
#     Nézd meg a patch tartalmát:
#       grep -E 'rename from|copy from|GIT binary patch|^diff -c ' <patch>
#     Ha bármelyik találat van: a fuzz nem segít, a patchet ki kell
#     venni vagy kézzel át kell írni. Csak ha üres a találat, menj
#     tovább a 3. lépésre. (Ez a leggyakrabban elfelejtett ellenőrzés.)
#
#  3. LÉPÉS – Nézd meg a debug-riportot
#     mpatch-debug-report-<timestamp>.md a ${srcdir}/mpatch-debug/-ben.
#     Tartalmazza:
#       - az eredeti patch-et
#       - a célfájl érintett részeit
#       - inline word-level diffet a nem illeszkedő hunkról
#       - a legközelebbi hasonló fájlútvonalakat (ha a fájl nem található)
#
#  4. LÉPÉS – Próbáld lazítani a fuzz-factort
#     Szerkeszd a PKGBUILD-ben a _mpatch_opts-ot:
#       --fuzz-factor 0.5
#     Futtasd újra: makepkg -f
#     Ha sikerül: a patch elavult, jegyezd fel, és később frissítsd.
#     Ha nem sikerül: 5. lépés.
#
#  5. LÉPÉS – Reverse-engineer
#     Nézd meg, mit csinál a patch eredetileg:
#       mpatch -R --dry-run <patch> kernel/
#     Ez megmutatja, mit venne ki a forrásból.
#     Nézd meg a Debian upstream git history-t:
#       https://salsa.debian.org/nvidia-team/nvidia-graphics-drivers
#     Keress rá a patch nevére, nézd meg az eredeti commitot.
#
#  6. LÉPÉS – Kihagyás vagy frissítés
#     Ha a patch már upstream-ben van: vedd ki a series.resolved-ból.
#     Ha a patch elavult: frissítsd a Debian salsa repóból.
#     Ha muszáj: írj saját patchet a különbségre.
#
#  ---------------------------------------------------------------------------
#  TESZTELÉSI TIPP
#  ---------------------------------------------------------------------------
#  Egyetlen patch tesztelése a teljes sorozat helyett:
#     cd ${srcdir}/NVIDIA-Linux-x86_64-470.256.02/kernel
#     mpatch --dry-run --fuzz-factor 0.7 ${srcdir}/<patch> .
#  Ez megmutatja, mit tenne, anélkül hogy bármit írna.
#
# =============================================================================


pkgbase=nvidia-470xx-utils
pkgname=('nvidia-470xx-utils' 'opencl-nvidia-470xx' 'nvidia-470xx-dkms' 'mhwd-nvidia-470xx')
pkgver=470.256.02
pkgrel=1
arch=('x86_64')
url="https://www.nvidia.com/"
license=('custom')
options=('!strip')
makedepends=('mpatch-git' 'nvidia-driver-assistant' 'patchelf')
_pkg="NVIDIA-Linux-x86_64-${pkgver}"

# =============================================================================
# MPATCH KONFIGURÁCIÓ
# =============================================================================

_mpatch_opts=(
    --atomic
)

_mpatch_debug_opts=(
    -vvvv
    --dry-run
)

_mpatch_debug_on_failure=1

# ---- Debian patch lista (a series.resolved-ból, sorrend kötelező) ----

_debian_patches=(
    cc_version_check-gcc5.patch
    bashisms.patch
    0001-some-power-management-features-were-not-yet-in-Linux.patch
    0023-backport-vm_area_struct_has_const_vm_flags-changes-f.patch
    0033-refuse-to-load-legacy-module-if-IBT-is-enabled.patch
    0034-fix-typos.patch
    0037-import-pfn_valid-w-o-GPL-rcu_read_lock-unlock-from-v.patch
    0042-Log-an-error-message-when-nv_mem_client_init-fails-d.patch
    0043-backport-follow_pfn-changes-from-550.90.07.patch
    0045-let-the-virt_addr_valid-macro-use-nv_pfn_valid-on-pp.patch
    0046-backport-nv_get_kern_phys_address-changes-from-555.4.patch
    0047-backport-drm_output_poll_changed-changes-from-535.21.patch
    0048-backport-cmd_symlink-changes-from-550.142.patch
    0049-backport-uvm-warning-fixes-from-510.39.01.patch
    0049-backport-warning-fixes-from-525.53.patch
    0050-backport-uvm-warning-fixes-from-535.146.02.patch
    0051-backport-warning-fixes-from-535.216.01.patch
    0052-backport-uvm-warning-fixes-from-560.28.03.patch
    0053-fix-more-warnings.patch
    0054-backport-warning-fixes-from-575.51.02.patch
    0054-fix-more-uvm-warnings.patch
    0055-backport-file_operations_fop_unsigned_offset_present.patch
    0056-backport-LD_SCRIPT-changes-from-535.230.02.patch
    0057-backport-uvm-fixes-from-535.230.02.patch
    0058-backport-warning-fixes-from-565.57.01.patch
    0059-backport-uvm-warning-fixes-from-550.90.07.patch
    0060-backport-build_cflags-changes-from-525.85.05.patch
    0061-backport-drm_gem_object_vmap_has_map_arg-changes-fro.patch
    0063-backport-conftest.sh-comment-changes-from-515.48.07.patch
    0063-backport-conftest.sh-comment-changes-from-525.53.patch
    0063-backport-conftest.sh-comment-changes-from-545.23.06.patch
    0064-backport-drm_driver_has_date-from-570.124.04.patch
    0065-backport-ccflags-y-changes-from-570.153.02.patch
    0066-backport-nv_timer_delete_sync-changes-from-570.153.0.patch
    0067-backport-drm_connector_helper_funcs_mode_valid_has_c.patch
    0071-backport-nv_vma_start_write-changes-from-570.169.patch
    0074-backport-drm_fb_create_takes_format_info-changes-fro.patch
    0075-backport-drm_print.h-changes-from-570.211.01.patch
    0076-backport-nv_in_hardirq-changes-from-580.119.02.patch
    0077-backport-vma_flags_set_word-changes-from-580.126.09.patch
    0078-backport-vma_flags_set_word-changes-from-580.126.09-.patch
    0079-backport-dma_map_ops_has_map_phys-changes-from-580.1.patch
    0080-backport-for_each_-_connector_in_state-changes-from-.patch
    0081-backport-for_each_-_crtc_in_state-changes-from-535.3.patch
    0082-backport-for_each_-_plane_in_state-changes-from-535..patch
    0083-backport-dma_fence_signal-changes-from-535.309.01.patch
    0084-backport-is_vma_write_locked_has_mm_lock_seq_arg-cha.patch
    0086-backport-acpi-changes-from-510.85.02.patch
    0087-backport-drm_atomic_commit_struct_present-changes-fr.patch
    0088-backport-strncpy-changes-from-580.178.04.patch
    0089-backport-strncpy-changes-from-580.178.04-uvm-part.patch
    0090-backport-MODULE_DESCRIPTION-changes-from-580.159.03.patch
    0091-backport-MODULE_DESCRIPTION-changes-from-580.159.03-.patch
    conftest-verbose.patch
    use-kbuild-compiler.patch
    use-kbuild-flags.patch
    conftest-prefer-arch-headers.patch
)

# =============================================================================
# FORRÁSOK
# =============================================================================

source=(
    '10-amdgpu-nvidia-drm-outputclass.conf'
    '10-intel-nvidia-drm-outputclass.conf'
    '90-nvidia-470xx-utils.hook'
    'nvidia-470xx-utils.install'
    'nvidia-470xx-utils.sysusers'
    'nvidia-470xx.rules'
    'nvidia-470xx-utils.conf'
    'systemd-homed-override.conf'
    'systemd-suspend-override.conf'
    "https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/${_pkg}.run"
    'series.resolved'
    "${_debian_patches[@]}"
)

# Minden forrás SHA256 ellenőrzése kikapcsolva, a kérésed szerint.
sha256sums=(
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
)

# =============================================================================
# SYMLINK GENERÁTOR
# =============================================================================

create_links() {
    find "$pkgdir" -type f -name '*.so*' ! -path '*xorg/*' -print0 |
    while IFS= read -r -d $'\0' _lib; do
        _soname=$(dirname "${_lib}")/$(readelf -d "${_lib}" |
            grep -Po 'SONAME.*: \[\K[^]]*' || true)

        [ -n "${_soname}" ] || continue

        _base=$(echo "${_soname}" | sed -r 's/(.*)\.so.*/\1.so/')

        [[ -e "${_soname}" ]] ||
            ln -s "$(basename "${_lib}")" "${_soname}"

        [[ -e "${_base}" ]] ||
            ln -s "$(basename "${_soname}")" "${_base}"
    done
}

# =============================================================================
# MPATCH WRAPPEREK
# =============================================================================

_mpatch_apply() {
    local _patchfile="$1"
    local _targetdir="$2"
    local _patchname
    local _rc

    _patchname=$(basename "$_patchfile")

    if [ ! -s "$_patchfile" ]; then
        echo "!!! HIBA: üres patchfájl: $_patchname" >&2
        return 1
    fi

    mpatch "${_mpatch_opts[@]}" "$_patchfile" "$_targetdir"
    _rc=$?

    if [ "$_rc" -eq 0 ]; then
        return 0
    fi

    echo "!!! mpatch FAILED (rc=$_rc): $_patchname" >&2

    if [ "${_mpatch_debug_on_failure}" = "1" ]; then
        local _dbgdir="${srcdir}/mpatch-debug"

        mkdir -p "$_dbgdir"

        (
            cd "$_targetdir" &&
            mpatch "${_mpatch_debug_opts[@]}" "$_patchfile" .
        ) > "$_dbgdir/${_patchname}.stdout" 2>&1 || true

        find "${_targetdir}" -maxdepth 1 \
            -name 'mpatch-debug-report-*.md' \
            -exec mv {} "$_dbgdir/${_patchname}.report.md" \; \
            2>/dev/null || true

        echo ">>> Debug riport: $_dbgdir/${_patchname}.report.md" >&2
    fi

    return "$_rc"
}

# =============================================================================
# PREPARE
# =============================================================================

prepare() {
    rm -rf "${_pkg}"

    sh "${_pkg}.run" --extract-only

    cd "${_pkg}"

    bsdtar -xf nvidia-persistenced-init.tar.bz2

    # NVIDIA 470: a telepítő már a kész Vulkan ICD JSON-t tartalmazza.
    if [ ! -f nvidia_icd.json ]; then
        echo "!!! HIBA: hiányzik az NVIDIA telepítőből a nvidia_icd.json"
        exit 1
    fi

    cd kernel

    if [ ! -f "${srcdir}/series.resolved" ]; then
        echo "!!! HIBA: hiányzik a series.resolved"
        exit 1
    fi

    # Ellenőrzés: a PKGBUILD tömbje és a series.resolved azonos legyen.
    local _expected
    local _actual

    _expected=$(printf '%s\n' "${_debian_patches[@]}")
    _actual=$(sed '/^[[:space:]]*$/d' "${srcdir}/series.resolved")

    if [ "$_expected" != "$_actual" ]; then
        echo "!!! HIBA: a _debian_patches és a series.resolved eltér!"
        exit 1
    fi

    local _n=0
    local _total
    local _patch
    local _patchfile

    _total=${#_debian_patches[@]}

    echo ">>> Debian patch-sorozat alkalmazása mpatch-csel"
    echo ">>> Patch-ek száma: $_total"
    echo ">>> Opciók: ${_mpatch_opts[*]}"

    while IFS= read -r _patch || [ -n "$_patch" ]; do
        _patch="${_patch%$'\r'}"
        [ -z "${_patch//[[:space:]]/}" ] && continue

        _n=$((_n + 1))
        _patchfile="${srcdir}/${_patch}"

        if [ ! -f "$_patchfile" ]; then
            echo "!!! HIBA: hiányzó patch: $_patchfile"
            exit 1
        fi

        if [ ! -s "$_patchfile" ]; then
            echo "!!! HIBA: üres patch: $_patch"
            exit 1
        fi

        printf '[%3d/%d] %s\n' "$_n" "$_total" "$_patch"

        _mpatch_apply "$_patchfile" . || {
            echo "!!! PATCH FAILED: $_patch"
            exit 1
        }
    done < "${srcdir}/series.resolved"

    echo ">>> Mind a $_n patch sikeresen alkalmazva."

    # DKMS konfiguráció
    sed -i "s/__VERSION_STRING/${pkgver}/" dkms.conf
    sed -i 's/__JOBS/`nproc`/' dkms.conf
    sed -i 's/__DKMS_MODULES//' dkms.conf

    sed -i '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"\
BUILT_MODULE_NAME[4]="nvidia-peermem"\
DEST_MODULE_LOCATION[4]="/kernel/drivers/video"' dkms.conf

    # PREEMPT_RT támogatás
    sed -i \
        's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' \
        dkms.conf

    cd ..
}

# =============================================================================
# OPENCL
# =============================================================================

package_opencl-nvidia-470xx() {
    pkgdesc="OpenCL implementation for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=("opencl-nvidia=${pkgver}" 'opencl-driver')
    conflicts=('opencl-nvidia')

    cd "${_pkg}"

    install -Dm644 nvidia.icd \
        "${pkgdir}/etc/OpenCL/vendors/nvidia.icd"

    install -Dm755 "libnvidia-compiler.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-compiler.so.${pkgver}"

    install -Dm755 "libnvidia-opencl.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-opencl.so.${pkgver}"

    create_links

    install -d "${pkgdir}/usr/share/licenses"
    ln -s nvidia-utils "${pkgdir}/usr/share/licenses/opencl-nvidia"
}

# =============================================================================
# NVIDIA DKMS
# =============================================================================

package_nvidia-470xx-dkms() {
    pkgdesc="NVIDIA 470xx driver sources for DKMS"
    depends=('dkms' "nvidia-470xx-utils=${pkgver}" 'libglvnd')
    provides=('NVIDIA-MODULE' "nvidia-dkms=${pkgver}")
    conflicts=('nvidia-dkms')

    cd "${_pkg}"

    install -dm755 "${pkgdir}/usr/src"

    cp -dr --no-preserve='ownership' kernel \
        "${pkgdir}/usr/src/nvidia-${pkgver}"

    install -Dm644 LICENSE \
        "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

# =============================================================================
# NVIDIA UTILITIES
# =============================================================================

package_nvidia-470xx-utils() {
    pkgdesc="NVIDIA 470xx drivers utilities"
    depends=('xorg-server' 'libglvnd' 'egl-wayland')
    optdepends=(
        "nvidia-settings=${pkgver}: configuration tool"
        'xorg-server-devel: nvidia-xconfig'
        "opencl-nvidia-470xx: OpenCL support"
    )

    provides=(
        'vulkan-driver'
        'opengl-driver'
        'nvidia-libgl'
        "nvidia-utils=${pkgver}"
    )

    conflicts=('nvidia-libgl' 'nvidia-utils')
    install="${pkgname}.install"

    cd "${_pkg}"

    # X driver
    install -Dm755 nvidia_drv.so \
        "${pkgdir}/usr/lib/xorg/modules/drivers/nvidia_drv.so"

    # GSP firmware
    install -Dm644 firmware/gsp.bin \
        "${pkgdir}/usr/lib/firmware/nvidia/${pkgver}/gsp.bin"

    # GLX extension module for X
    install -Dm755 "libglxserver_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.${pkgver}"

    ln -s "libglxserver_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.1"
    ln -s "libglxserver_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so"

    install -Dm755 "libGLX_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -Dm755 "libEGL_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/libEGL_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv1_CM_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/libGLESv1_CM_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv2_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/libGLESv2_nvidia.so.${pkgver}"
    install -Dm644 10_nvidia.json \
        "${pkgdir}/usr/share/glvnd/egl_vendor.d/10_nvidia.json"

    # OpenGL core libraries
    install -Dm755 "libnvidia-glcore.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-glcore.so.${pkgver}"
    install -Dm755 "libnvidia-eglcore.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-eglcore.so.${pkgver}"
    install -Dm755 "libnvidia-glsi.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-glsi.so.${pkgver}"

    # Miscellaneous NVIDIA libraries
    install -Dm755 "libnvidia-ifr.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-ifr.so.${pkgver}"
    install -Dm755 "libnvidia-fbc.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-fbc.so.${pkgver}"
    install -Dm755 "libnvidia-encode.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-encode.so.${pkgver}"
    install -Dm755 "libnvidia-cfg.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-cfg.so.${pkgver}"
    install -Dm755 "libnvidia-ml.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-ml.so.${pkgver}"
    install -Dm755 "libnvidia-glvkspirv.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-glvkspirv.so.${pkgver}"
    install -Dm755 "libnvidia-allocator.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}"
    install -Dm755 "libnvidia-vulkan-producer.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.${pkgver}"

    # The Vulkan producer library has no SONAME in the NVIDIA binary.
    patchelf --set-soname "libnvidia-vulkan-producer.so.1" \
        "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.${pkgver}"

    # Vulkan ICD and implicit layer
    install -Dm644 nvidia_icd.json \
        "${pkgdir}/usr/share/vulkan/icd.d/nvidia_icd.json"
    install -Dm644 nvidia_layers.json \
        "${pkgdir}/usr/share/vulkan/implicit_layer.d/nvidia_layers.json"

    # VDPAU
    install -Dm755 "libvdpau_nvidia.so.${pkgver}" \
        "${pkgdir}/usr/lib/vdpau/libvdpau_nvidia.so.${pkgver}"

    # NVIDIA TLS library (the file is in the archive root, not tls/).
    install -Dm755 "libnvidia-tls.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -Dm755 "libcuda.so.${pkgver}" \
        "${pkgdir}/usr/lib/libcuda.so.${pkgver}"
    install -Dm755 "libnvcuvid.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvcuvid.so.${pkgver}"

    # PTX JIT compiler
    install -Dm755 "libnvidia-ptxjitcompiler.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-ptxjitcompiler.so.${pkgver}"

    # Ray tracing
    install -Dm755 "libnvoptix.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvoptix.so.${pkgver}"
    install -Dm755 "libnvidia-rtcore.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-rtcore.so.${pkgver}"
    install -Dm755 "libnvidia-cbl.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-cbl.so.${pkgver}"

    # NGX
    install -Dm755 nvidia-ngx-updater \
        "${pkgdir}/usr/bin/nvidia-ngx-updater"
    install -Dm755 "libnvidia-ngx.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-ngx.so.${pkgver}"
    install -Dm755 _nvngx.dll \
        "${pkgdir}/usr/lib/nvidia/wine/_nvngx.dll"
    install -Dm755 nvngx.dll \
        "${pkgdir}/usr/lib/nvidia/wine/nvngx.dll"

    # Optical flow
    install -Dm755 "libnvidia-opticalflow.so.${pkgver}" \
        "${pkgdir}/usr/lib/libnvidia-opticalflow.so.${pkgver}"

    # Debug utility
    install -Dm755 nvidia-debugdump \
        "${pkgdir}/usr/bin/nvidia-debugdump"

    # nvidia-xconfig
    install -Dm755 nvidia-xconfig \
        "${pkgdir}/usr/bin/nvidia-xconfig"
    install -Dm644 nvidia-xconfig.1.gz \
        "${pkgdir}/usr/share/man/man1/nvidia-xconfig.1.gz"

    # nvidia-bug-report
    install -Dm755 nvidia-bug-report.sh \
        "${pkgdir}/usr/bin/nvidia-bug-report.sh"

    # nvidia-smi
    install -Dm755 nvidia-smi \
        "${pkgdir}/usr/bin/nvidia-smi"
    install -Dm644 nvidia-smi.1.gz \
        "${pkgdir}/usr/share/man/man1/nvidia-smi.1.gz"

    # nvidia-cuda-mps
    install -Dm755 nvidia-cuda-mps-server \
        "${pkgdir}/usr/bin/nvidia-cuda-mps-server"
    install -Dm755 nvidia-cuda-mps-control \
        "${pkgdir}/usr/bin/nvidia-cuda-mps-control"
    install -Dm644 nvidia-cuda-mps-control.1.gz \
        "${pkgdir}/usr/share/man/man1/nvidia-cuda-mps-control.1.gz"

    # nvidia-modprobe
    install -Dm4755 nvidia-modprobe \
        "${pkgdir}/usr/bin/nvidia-modprobe"
    install -Dm644 nvidia-modprobe.1.gz \
        "${pkgdir}/usr/share/man/man1/nvidia-modprobe.1.gz"

    # nvidia-persistenced
    install -Dm755 nvidia-persistenced \
        "${pkgdir}/usr/bin/nvidia-persistenced"
    install -Dm644 nvidia-persistenced.1.gz \
        "${pkgdir}/usr/share/man/man1/nvidia-persistenced.1.gz"
    install -Dm644 nvidia-persistenced-init/systemd/nvidia-persistenced.service.template \
        "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"
    sed -i 's/__USER__/nvidia-persistenced/' \
        "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"

    # NVIDIA application profiles
    install -Dm644 "nvidia-application-profiles-${pkgver}-rc" \
        "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-rc"
    install -Dm644 "nvidia-application-profiles-${pkgver}-key-documentation" \
        "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-key-documentation"

    # License and documentation
    install -Dm644 LICENSE \
        "${pkgdir}/usr/share/licenses/nvidia-470xx-utils/LICENSE"
    install -Dm644 README.txt \
        "${pkgdir}/usr/share/doc/nvidia/README"
    install -Dm644 NVIDIA_Changelog \
        "${pkgdir}/usr/share/doc/nvidia/NVIDIA_Changelog"
    cp -r html "${pkgdir}/usr/share/doc/nvidia/"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-utils"

    # NVIDIA systemd power-management support
    install -Dm644 systemd/system/nvidia-suspend.service \
        "${pkgdir}/usr/lib/systemd/system/nvidia-suspend.service"
    install -Dm644 systemd/system/nvidia-hibernate.service \
        "${pkgdir}/usr/lib/systemd/system/nvidia-hibernate.service"
    install -Dm644 systemd/system/nvidia-resume.service \
        "${pkgdir}/usr/lib/systemd/system/nvidia-resume.service"
    install -Dm755 systemd/system-sleep/nvidia \
        "${pkgdir}/usr/lib/systemd/system-sleep/nvidia"
    install -Dm755 systemd/nvidia-sleep.sh \
        "${pkgdir}/usr/bin/nvidia-sleep.sh"

    # systemd overrides
    install -Dm644 "${srcdir}/systemd-homed-override.conf" \
        "${pkgdir}/usr/lib/systemd/system/systemd-homed.service.d/10-nvidia-no-freeze-session.conf"
    install -Dm644 "${srcdir}/systemd-suspend-override.conf" \
        "${pkgdir}/usr/lib/systemd/system/systemd-suspend.service.d/10-nvidia-no-freeze-session.conf"
    install -Dm644 "${srcdir}/systemd-suspend-override.conf" \
        "${pkgdir}/usr/lib/systemd/system/systemd-suspend-then-hibernate.service.d/10-nvidia-no-freeze-session.conf"
    install -Dm644 "${srcdir}/systemd-suspend-override.conf" \
        "${pkgdir}/usr/lib/systemd/system/systemd-hibernate.service.d/10-nvidia-no-freeze-session.conf"
    install -Dm644 "${srcdir}/systemd-suspend-override.conf" \
        "${pkgdir}/usr/lib/systemd/system/systemd-hybrid-sleep.service.d/10-nvidia-no-freeze-session.conf"

    # Xorg outputclass configuration
    install -Dm644 "${srcdir}/10-amdgpu-nvidia-drm-outputclass.conf" \
        "${pkgdir}/usr/share/X11/xorg.conf.d/10-amdgpu-nvidia-drm-outputclass.conf"
    install -Dm644 "${srcdir}/10-intel-nvidia-drm-outputclass.conf" \
        "${pkgdir}/usr/share/X11/xorg.conf.d/10-intel-nvidia-drm-outputclass.conf"

    # sysusers
    install -Dm644 "${srcdir}/nvidia-470xx-utils.sysusers" \
        "${pkgdir}/usr/lib/sysusers.d/${pkgname}.conf"

    # udev rules
    install -Dm644 "${srcdir}/nvidia-470xx.rules" \
        "${pkgdir}/usr/lib/udev/rules.d/60-nvidia-470xx.rules"

    # modprobe configuration
    install -Dm644 "${srcdir}/nvidia-470xx-utils.conf" \
        "${pkgdir}/usr/lib/modprobe.d/nvidia-470xx-utils.conf"

    # ALPM hook
    install -Dm644 "${srcdir}/90-nvidia-470xx-utils.hook" \
        "${pkgdir}/usr/share/libalpm/hooks/90-nvidia-470xx-utils.hook"

    create_links
}
# =============================================================================
# MHWD ID GENERÁLÁS
# =============================================================================

package_mhwd-nvidia-470xx() {
    pkgdesc="MHWD module-ids for nvidia ${pkgver}"
    arch=('any')
    depends=('mhwd')

    install -d -m755 "${pkgdir}/var/lib/mhwd/ids/pci/"

    nvidia-driver-assistant --generate-ids 470 \
        > "${pkgdir}/var/lib/mhwd/ids/pci/nvidia-470xx.ids"
}
