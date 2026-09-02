# syntax=docker/dockerfile:1-labs
FROM cachyos/cachyos-v3 AS builder

# 0. Initialize and populate the pacman keyring
RUN pacman-key --init && \
    pacman-key --populate archlinux cachyos

# 1. Configure pacman to aggressively skip docs, locales, and disable sandbox
RUN sed -i 's|^#NoExtract.*|NoExtract = usr/share/help/* usr/share/gtk-doc/* usr/share/doc/* usr/share/man/* usr/share/info/* usr/share/locale/*|' /etc/pacman.conf && \
    sed -i '/^\[options\]/a DisableSandbox\nDisableSandboxNetwork' /etc/pacman.conf && \
    echo 'PACMAN_OPTS=("--disable-sandbox")' >> /etc/makepkg.conf

# 2. Setup minimal cachyos-rate-mirrors
RUN pacman -Syu --noconfirm --needed cachyos-rate-mirrors base-devel git sudo && \
    export RATE_MIRRORS_PROTOCOL=https \
    RATE_MIRRORS_ALLOW_ROOT=true \
    RATE_MIRRORS_DISABLE_COMMENTS=true \
    RATE_MIRRORS_FETCH_MIRRORS_TIMEOUT=5000 \
    RATE_MIRRORS_PER_MIRROR_TIMEOUT=5000 \
    RATE_MIRRORS_MAX_MIRRORS_TO_OUTPUT=30 && \
    rate-mirrors arch | tee /etc/pacman.d/mirrorlist

WORKDIR /rootfs

# 3. Setup minimal directory structure required by pacman
RUN mkdir -p etc/pacman.d var/lib/pacman var/cache/pacman/pkg usr/bin usr/lib usr/share etc/ssl

# 4. Copy configs, mirrorlists, and trust anchors (certificates) from the builder
RUN cp /etc/pacman.conf ./etc/pacman.conf && \
    cp -r /etc/pacman.d/* ./etc/pacman.d/ && \
    cp -r /etc/pacman.d/gnupg ./etc/pacman.d/ 2>/dev/null || true && \
    cp -r /etc/ssl/certs ./etc/ssl/ && \
    cp -r /etc/ca-certificates ./etc/

# 5. Bootstrap the balanced x86-64-v3 runtime
RUN pacman -Sy --root /rootfs --config /rootfs/etc/pacman.conf --dbpath /rootfs/var/lib/pacman --noconfirm --overwrite '*' \
    filesystem \
    bash \
    coreutils \
    git \
    pacman \
    paru \
    yay \
    cachyos-v3-mirrorlist \
    cachyos-mirrorlist \
    cachyos-keyring \
    archlinux-keyring \
    cachyos-rate-mirrors \
    glibc \
    libstdc++ \
    libgcc \
    curl \
    wget \
    ca-certificates \
    iproute2 \
    iputils \
    procps-ng \
    util-linux \
    tar \
    unzip \
    gzip \
    bzip2 \
    xz \
    zstd \
    less \
    which \
    file \
    sed \
    grep \
    gawk \
    findutils \
    diffutils \
    make \
    patch \
    pkgconf \
    binutils \
    fakeroot \
    sudo \
    shadow

# 6. Create 'app' user and configure password-less sudo
RUN chroot /rootfs /usr/bin/useradd -m -s /usr/bin/bash app && \
    echo "app ALL=(ALL:ALL) NOPASSWD: ALL" >> /rootfs/etc/sudoers

# 7. Aggressive Cleanup & Pruning inside the new rootfs
RUN rm -rf \
    /rootfs/var/cache/pacman/pkg/* \
    /rootfs/var/lib/pacman/sync/* \
    /rootfs/usr/share/{man,doc,info,locale,gtk-doc,help}/* \
    /rootfs/var/log/* \
    /rootfs/tmp/* \
    /rootfs/var/tmp/* && \
    find /rootfs/etc -name "*.pacnew" -type f -delete

# ==============================================================================
# STAGE 2: Balanced x86-64-v3 Runtime
# ==============================================================================
FROM scratch

# 1. Copy the pruned filesystem from the builder stage
COPY --from=builder /rootfs /

# 2. Set essential environment variables
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    ARCH=x86_64_v3 \
    HOSTNAME=archy

# 3. Ensure shell defaults to CachyOS optimized bash
SHELL ["/usr/bin/bash", "-c"]

# 4. Switch to the non-root app user and set the home directory
USER app
WORKDIR /home/app

# 5. Set a custom shell prompt
ENV PS1="\[\e[0;32m\]\u@archy\[\e[0m\] \[\e[0;32m\]\w\[\e[0m\]# "

# 6. Build zlib-ng-compat-git and replace core:zlib
RUN sudo pacman -Sy --noconfirm && \
    mkdir -p zlibng && \
    cd zlibng && \
    curl -sLO 'https://raw.githubusercontent.com/rokibhasansagar/Arch_PKGBUILDs/refs/heads/main/zlib-ng-compat-git/PKGBUILD' && \
    makepkg -sri --noconfirm && \
    cd .. && \
    zlibng_ver=$(pacman -Q zlib | awk '{print $2}') && \
    sudo sed -i '/^HoldPkg\s*=/ s/$/ zlib/' /etc/pacman.conf && \
    sudo sed -i '/^#IgnorePkg/c\IgnorePkg = zlib' /etc/pacman.conf && \
    sudo sed -i '/^#AssumeInstalled/c\AssumeInstalled = zlib='"${zlibng_ver}"'' /etc/pacman.conf && \
    sudo rm -rf ./zlibng /var/lib/pacman/sync/* /var/cache/pacman/pkg/*

# 7. List installed packages for debug purpose
RUN echo -e "[+] PreInstalled Packages:" && \
    sudo pacman -Q 2>/dev/null

# 8. Default entry point
CMD ["/usr/bin/bash"]

