# syntax=docker/dockerfile:1-labs
FROM cachyos/cachyos-v3 AS builder

RUN sudo pacman -Syu --noconfirm --needed cachyos-rate-mirrors

WORKDIR /rootfs

# 1. Setup minimal directory structure required by pacman (Added etc/ssl for certs)
RUN mkdir -p etc/pacman.d var/lib/pacman var/cache/pacman/pkg usr/bin usr/lib usr/share etc/ssl

# 2. Configure pacman to aggressively skip docs, man pages, locales, and disable sandbox
RUN sed -i 's|^#NoExtract.*|NoExtract = usr/share/help/* usr/share/gtk-doc/* usr/share/doc/* usr/share/man/* usr/share/info/* usr/share/locale/*|' /etc/pacman.conf && \
    sed -i '/^\[options\]/a DisableSandbox' /etc/pacman.conf

# 3. Copy configs, mirrorlists, and trust anchors (certificates) from the builder
RUN cp /etc/pacman.conf ./etc/pacman.conf && \
    cp -r /etc/pacman.d/* ./etc/pacman.d/ && \
    cp -r /etc/pacman.d/gnupg ./etc/pacman.d/ 2>/dev/null || true && \
    cp -r /etc/ssl/certs ./etc/ssl/ && \
    cp -r /etc/ca-certificates ./etc/

# 4. Bootstrap the balanced x86-64-v3 runtime (Added --overwrite '*' for cert replacement)
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
    sudo \
    shadow

# 5. Initialize and populate the pacman keyring for the new filesystem
RUN chroot /rootfs /usr/bin/pacman-key --init && \
    chroot /rootfs /usr/bin/pacman-key --populate archlinux cachyos

# 5.xyz: Various fixes
RUN mkdir -p zlib-ng-compat-git && \
    cd zlib-ng-compat-git && \
    curl -sLO "https://raw.githubusercontent.com/rokibhasansagar/Arch_PKGBUILDs/refs/heads/main/zlib-ng-compat-git/PKGBUILD" && \
    chroot /rootfs /usr/bin/paru -Ui --noconfirm --needed --skipreview --noprovides --useask --combinedupgrade --batchinstall --noinstalldebug --removemake --nocleanafter --nokeepsrc --mflags="--force" --rebuild && \
    cd .. && \
    rm -rf ./zlib-ng-compat-git && \
    zlibng_ver="1:1.3.2-3.1" && \
    sed -i '/^#AssumeInstalled/c\AssumeInstalled = zlib='"${zlibng_ver}"'' /etc/pacman.conf

# 6. Create 'app' user and configure password-less sudo
# We use chroot to run useradd inside the newly created filesystem context.
RUN chroot /rootfs /usr/bin/useradd -m -s /usr/bin/bash app && \
    echo "app ALL=(ALL:ALL) NOPASSWD: ALL" >> /rootfs/etc/sudoers

# 7. Aggressive Cleanup & Pruning inside the new rootfs (Added .pacnew cleanup)
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

# Copy the pruned filesystem from the builder stage
COPY --from=builder /rootfs /

# Set essential environment variables
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    ARCH=x86_64_v3 \
    HOSTNAME=archy

# Ensure shell defaults to CachyOS optimized bash
SHELL ["/usr/bin/bash", "-c"]

# Switch to the non-root app user and set the home directory
USER app
WORKDIR /home/app

# Set a custom shell prompt
ENV PS1="\[\e[0;32m\]\u@archy\[\e[0m\] \[\e[0;32m\]\w\[\e[0m\]# "

# Default entry point
CMD ["/usr/bin/bash"]

