# syntax=docker/dockerfile:1-labs

# Catch ARGs to determine which base image to pull
ARG TARGET=v3
ARG ARCH=x86_64_v3

# Dynamically link the correct base image suffix (e.g., -v3, -v4)
FROM fr3akyphantom/cachy-base-${TARGET}:latest AS builder

# Redeclare ARGs inside the builder
ARG ARCH

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV SHELL=/bin/bash

# 0. Initialize and populate the pacman keyring
RUN sudo pacman-key --init && \
    sudo pacman-key --populate archlinux cachyos

RUN <<-'EOS'
	set -xe
	# Update System Immediately \
	sudo pacman -Syu --noconfirm 2>/dev/null
	# re-Install base-devel \
	sudo pacman -S --noconfirm --needed base-devel
	echo -e "[+] List of Packages Before Actual Operation:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
	echo -e "[+] Build Tools PreInstallation"
	paru -S --noconfirm --needed ${PARU_OPTS} \
	cmake ninja clang yasm nasm meson compiler-rt jq rust cargo-c libgit2 zip p7zip python-pip python-utils
EOS

RUN <<-'EOT'
	set -xe
	echo -e "[+] List of Packages Before Installing Dependency Apps:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
	mkdir -p /home/app/.cache/paru/clone 2>/dev/null
	git clone -q --filter=blob:none https://github.com/rokibhasansagar/Arch_PKGBUILDs.git /home/app/.cache/paru/pkgbuilds/
	rm -rf /home/app/.cache/paru/pkgbuilds/.git
	_custPKGBuilder() {
	  # 1. Translate Docker's ARCH variable to compiler format
	  # (Converts 'x86_64_v4' to 'x86-64-v4')
	  local TARGET_CPU="${ARCH:-native}"
	  TARGET_CPU="${TARGET_CPU//_/-}"
	  # 2. Dynamically generate a makepkg.conf override.
	  # By appending to ${CFLAGS}, CachyOS's default -O3 and LTO flags are preserved,
	  # but our highly specific -march flag gets appended at the end (overriding base arches).
	  echo "CFLAGS=\"\${CFLAGS} -march=${TARGET_CPU}\"" > /home/app/.makepkg.conf
	  echo "CXXFLAGS=\"\${CXXFLAGS} -march=${TARGET_CPU}\"" >> /home/app/.makepkg.conf
	  echo "RUSTFLAGS=\"-C target-cpu=${TARGET_CPU} \${RUSTFLAGS}\"" >> /home/app/.makepkg.conf
	  for pkg in "${pkgs[@]}"; do
	    echo -e "[+] ${pkg} Build+Installation with makepkg"
	    cd /home/app/.cache/paru/pkgbuilds/${pkg}/
	    paru -Ui --noconfirm --needed --nodevel ${PARU_OPTS} --mflags="--force" --rebuild
	  done
	  # 3. Clean up the override so it doesn't leave lingering state
	  rm -f /home/app/.makepkg.conf
	}
	_uninstPKG() {
	  for pkg in "${unpkgs[@]}"; do
	    sudo pacman -Rdd ${pkg} --noconfirm 2>/dev/null || true
	  done
	}
	echo -e "[+] vapoursynth, ffmpeg and other tools Installation with pacman"
	paru -S --noconfirm --needed ${PARU_OPTS} ffmpeg ffms2 mkvtoolnix-cli numactl uv vapoursynth onetbb
	export pkgs=(vapoursynth-api3-headers) && _custPKGBuilder
	sudo uv pip install --system --break-system-packages vapoursynth-lsmas vapoursynth-akarin nativeres soifunc kagefunc vsjetpack[full,gpu,vulkan] --extra-index-url https://jaded-encoding-thaumaturgy.github.io/vs-wheels/simple
	whereis vapoursynth || true
	whereis vspipe || true
	PLUGINDIR=$(python -c "import vapoursynth; print(vapoursynth.get_plugin_dir())")
	echo -e "PlugDir == ${PLUGINDIR}"
	libtool --finish "${PLUGINDIR}" &>/dev/null
	export pkgs=(av1an-git) && _custPKGBuilder
	export pkgs=(x265) && _uninstPKG
	export pkgs=(x265-git) && _custPKGBuilder
	echo "Testing VapourSynth lsmas Autoloading..."
	python3 -c "import vapoursynth; core = vapoursynth.core; plugins = [p.name for p in core.plugins()]; print('Successfully loaded plugins:', plugins); assert any('lsmash' in p.lower() for p in plugins), 'Plugin failed to autoload!'; print('✅ SUCCESS: Plugin autoloaded correctly!')"
	export pythondir=$(python -c "import os;print(os.path.dirname(os.__file__))")
	libtool --finish /usr/lib &>/dev/null
	libtool --finish ${pythondir}/site-packages &>/dev/null
	sudo ldconfig 2>/dev/null
	echo -e "[+] Plugins Installation Block Starts Here"
	# export pkgs=({vapoursynth-plugin-,}waifu2x-ncnn-vulkan-git) && _custPKGBuilder
	cd /tmp && ( sudo pacman -Rdd nvidia-utils --noconfirm 2>/dev/null || true ) && paru -S --noconfirm --needed ${PARU_OPTS} vulkan-swrast
	libtool --finish /usr/lib/vapoursynth &>/dev/null
	sudo ldconfig 2>/dev/null
	echo -e "[+] List of All Packages After Base Installation:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
	echo -e "[>] PacCache Investigation"
	sudo du -sh /var/cache/pacman/pkg /home/app/.cache/paru/*
	ls -lAog /var/cache/pacman/pkg/*.pkg.tar.zst 2>/dev/null
	echo -e "[i] Encoder and Av1an Investigation"
	( ffmpeg -hide_banner -version || true )
	( x265 -V 2>&1 || true )
	( aomenc --help | grep "AOMedia Project AV1 Encoder" || true )
	( vspipe --version || true )
	( rav1e --version || true )
	( av1an --version || true )
	( SvtAv1EncApp --version || true )
	ls -lAog /home/app/.cache/paru/pkgbuilds/*/*.pkg.tar.zst /var/cache/pacman/pkg/*.pkg.tar.zst
	echo -e "[<] Cleanup"
	find "${pythondir}" "${PLUGINDIR}" -depth -type d -name __pycache__ -exec sudo rm -rf '{}' + 2>/dev/null  # /usr/share/
	( sudo pacman -Rdd cmake ninja clang yasm meson rust cargo-c compiler-rt base-devel zlib-ng --noconfirm 2>/dev/null || true )
	sudo rm -rf /root/.cache /tmp/* /var/lib/pacman/sync/* /var/cache/pacman/pkg/* /home/app/.cache/zig/* /home/app/.cache/yay/* /home/app/.cache/paru/{clone,pkgbuilds}/* /home/app/.cargo/* 2>/dev/null
	echo -e "[+] List of All Packages At The End Of All Process:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
EOT

# ==============================================================================
# STAGE 2: Stripping & Processing (Fat)
# ==============================================================================
FROM scratch AS fat

# Redeclare ARG for this stage
ARG ARCH

# 1. Copy the pruned filesystem from the builder stage
COPY --from=builder / /

# 2. Re-establish ENVs so bash, find, and strip work properly in the RUN block below
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    ARCH=${ARCH} \
    HOSTNAME=archy

# Ensure shell defaults to CachyOS optimized bash
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV SHELL=/bin/bash

RUN <<-'EOX'
	safe_strip() {
	  local target_dir=$1
	  find "$target_dir" -type f | while read -r f; do
	    local file_info
	    file_info=$(file "$f")
	    # Check 1: Is it an ELF file that still contains strippable symbols?
	    if echo "$file_info" | grep -q -E "ELF.*not stripped"; then
	      # Check 2: Is it statically linked (like rclone)?
	      if echo "$file_info" | grep -q "statically linked"; then
	        # Safe to obliterate everything for static binaries
	        strip --strip-all "$f" 2>/dev/null || true
	      else
	        # Keep .dynsym for shared libraries and dynamically linked executables
	        strip --strip-unneeded "$f" 2>/dev/null || true
	      fi
	      echo "[-] Reduced size via strip: $f"
	    fi
	  done
	}
	df -hlT
	sudo rm -rf /root/.cache
	safe_strip /
	df -hlT
EOX

# ==============================================================================
# STAGE 3: Final Environment
# ==============================================================================
FROM scratch

# Redeclare ARG for the final runtime stage
ARG ARCH

# 1. Copy the flattened, stripped filesystem from the fat stage
COPY --from=fat / /

# 2. Set permanent runtime metadata natively
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    ARCH=${ARCH} \
    HOSTNAME=archy

# 3. Ensure shell defaults to CachyOS optimized bash
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV SHELL=/bin/bash

# 4. Switch to the non-root app user and set the home directory
USER app
WORKDIR /home/app

# Re-apply custom prompt
ENV PS1="\[\e[0;32m\]\u@archy\[\e[0m\] \[\e[0;32m\]\w\[\e[0m\]# "

CMD ["/bin/bash"]

