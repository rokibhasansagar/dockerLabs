# syntax=docker/dockerfile:1-labs
FROM fr3akyphantom/cachy-base

# 0. Initialize and populate the pacman keyring
RUN sudo pacman-key --init && \
    sudo pacman-key --populate archlinux cachyos

SHELL ["/bin/bash", "-c"]

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
	echo -e "[+] List of Packages Before Installing Dependency Apps:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
	mkdir -p /home/app/.cache/paru/clone 2>/dev/null
	git clone -q --filter=blob:none https://github.com/rokibhasansagar/Arch_PKGBUILDs.git /home/app/.cache/paru/pkgbuilds/
	rm -rf /home/app/.cache/paru/pkgbuilds/.git
	_custPKGBuilder() {
	  for pkg in "${pkgs[@]}"; do
	    echo -e "[+] ${pkg} Build+Installation with makepkg"
	    cd /home/app/.cache/paru/pkgbuilds/${pkg}/
	    ( paru -Ui --noconfirm --needed ${PARU_OPTS} --mflags="--force" --rebuild )
	  done
	}
	_uninstPKG() {
	  for pkg in "${unpkgs[@]}"; do
	    ( sudo pacman -Rdd ${pkg} --noconfirm 2>/dev/null ) || true
	  done
	}
	echo -e "[+] vapoursynth-git, ffmpeg and other tools Installation with pacman"
	paru -S --noconfirm --needed ${PARU_OPTS} ffmpeg ffms2 mkvtoolnix-cli numactl vapoursynth av1an uv
	# sudo find /usr -type f -name "EXTERNALLY-MANAGED" -delete
	pip install vapoursynth-lsmas --break-system-packages
	export pkgs=(vapoursynth-api3-headers) && _custPKGBuilder
	export pythondir=$(python -c "import os;print(os.path.dirname(os.__file__))")
	libtool --finish /usr/lib &>/dev/null
	libtool --finish ${pythondir}/site-packages &>/dev/null
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
	find "${pythondir}" -depth -type d -name __pycache__ -exec sudo rm -rf '{}' + 2>/dev/null  # /usr/share/
	( sudo pacman -Rdd cmake ninja clang yasm meson rust cargo-c compiler-rt --noconfirm 2>/dev/null || true )
	sudo rm -rf /tmp/* /var/lib/pacman/sync/* /var/cache/pacman/pkg/* /home/app/.cache/zig/* /home/app/.cache/yay/* /home/app/.cache/paru/{clone,pkgbuilds}/* /home/app/.cargo/* 2>/dev/null
	echo -e "[+] List of All Packages At The End Of All Process:"
	echo -e "$(sudo pacman -Q | awk '{print $1}' | sed -z 's/\n/ /g;s/\s$/\n/g')" 2>/dev/null
EOS

ENTRYPOINT [ "/usr/bin/bash" ]

