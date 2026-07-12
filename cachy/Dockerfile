# syntax=docker/dockerfile:1-labs
FROM cachyos/cachyos-v3 AS builder

WORKDIR /rootfs

# 1. Setup minimal directory structure required by pacman
RUN mkdir -p etc/pacman.d var/lib/pacman var/cache/pacman/pkg \
    usr/bin usr/lib usr/share

# 2. Copy the optimized CachyOS repository configs and mirrorlists from the builder
RUN cp /etc/pacman.conf ./etc/pacman.conf && \
    cp -r /etc/pacman.d/* ./etc/pacman.d/ && \
    cp -r /etc/pacman.d/gnupg ./etc/pacman.d/ 2>/dev/null || true

# 3. Configure pacman to aggressively skip docs, man pages, and locales
RUN sed -i '/^#NoExtract/d' ./etc/pacman.conf && \
    echo "NoExtract = usr/share/man/* usr/share/doc/* usr/share/locale/* usr/share/info/*" >> ./etc/pacman.conf

# 4. Bootstrap the balanced x86-64-v3 runtime
# Replaced gcc-libs with explicit libstdc++, glibc, and libgcc.
# Added practical on-the-go tools from 'base' and 'base-devel'.
RUN pacman -Sy --root /rootfs --dbpath /rootfs/var/lib/pacman --noconfirm \
    filesystem \
    bash \
    coreutils \
    pacman \
    cachyos-v3-mirrorlist \
    cachyos-mirrorlist \
    cachyos-keyring \
    archlinux-keyring \
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
    sudo

# 5. Aggressive Cleanup & Pruning inside the new rootfs
RUN rm -rf \
    /rootfs/var/cache/pacman/pkg/* \
    /rootfs/var/lib/pacman/sync/* \
    /rootfs/usr/share/{man,doc,info,locale,gtk-doc}/* \
    /rootfs/var/log/* \
    /rootfs/tmp/* \
    /rootfs/var/tmp/*

# ==============================================================================
# STAGE 2: Balanced x86-64-v3 Runtime
# ==============================================================================
FROM scratch

# Metadata
LABEL org.opencontainers.image.title="CachyOS x86-64-v3 Balanced Runtime" \
      org.opencontainers.image.description="Practical, optimized x86-64-v3 CachyOS runtime with essential CLI and build utilities"

# Copy the pruned filesystem from the builder stage
COPY --from=builder /rootfs /

# Set essential environment variables
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    ARCH=x86_64_v3 \
    HOSTNAME=archy

# Set a custom shell prompt
ENV PS1="\[\e[0;32m\]\u@archy\[\e[0m\] \[\e[0;32m\]\w\[\e[0m\]# "

# Ensure shell defaults to CachyOS optimized bash
SHELL ["/usr/bin/bash", "-c"]

# Default entry point
CMD ["/usr/bin/bash"]

