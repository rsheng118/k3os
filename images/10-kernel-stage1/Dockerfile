ARG REPO
ARG TAG
FROM ubuntu:jammy

RUN apt-get --assume-yes update \
 && apt-get --assume-yes install \
    curl \
    initramfs-tools \
    kmod \
    lz4 \
    rsync \
    xz-utils \
 && echo 'r8152' >> /etc/initramfs-tools/modules \
 && echo 'hfs' >> /etc/initramfs-tools/modules \
 && echo 'hfsplus' >> /etc/initramfs-tools/modules \
 && echo 'nls_utf8' >> /etc/initramfs-tools/modules \
 && echo 'nls_iso8859_1' >> /etc/initramfs-tools/modules



ARG ARCH
ENV KVERSION=5.15.0-37-generic

ENV KERNEL_FIRMWARE=linux-firmware
ENV KERNEL_HEADERS=linux-headers-5.15.0-37
ENV KERNEL_IMG=linux-image-5.15.0-37-generic
ENV KERNEL_MODULE=linux-modules-5.15.0-37-generic
ENV KERNEL_MODULE_EXTRA=linux-modules-extra-5.15.0-37-generic

# Download kernel
RUN mkdir -p /usr/src && cd /usr/src
RUN apt download ${KERNEL_HEADERS}
RUN apt download ${KERNEL_FIRMWARE}
RUN apt download ${KERNEL_IMG}
RUN apt download ${KERNEL_MODULE}
RUN apt download ${KERNEL_MODULE_EXTRA}

# Extract to /usr/src/root
RUN mkdir -p /usr/src/root && \
    cd /usr/src/root && \
    dpkg-deb -x /linux-firmware*.deb /usr/src/root/ && \
    dpkg-deb -x /linux-headers*.deb /usr/src/root/ && \
    dpkg-deb -x /linux-image*.deb /usr/src/root/ && \
    dpkg-deb -x /linux-modules-5*.deb /usr/src/root/ && \
    dpkg-deb -x /linux-modules-extra*.deb /usr/src/root/

# Create initrd
RUN mkdir /usr/src/initrd && \
    rsync -a /usr/src/root/lib/ /lib/ && \
    depmod $KVERSION && \
    mkinitramfs -k $KVERSION -c lz4 -o /usr/src/initrd.tmp

# Generate initrd firmware and module lists
RUN mkdir -p /output/lib && \
    mkdir -p /output/headers && \
    cd /usr/src/initrd && \
    lz4cat /usr/src/initrd.tmp | cpio -idmv && \
    find lib/modules -name \*.ko > /output/initrd-modules && \
    echo lib/modules/${KVERSION}/modules.order >> /output/initrd-modules && \
    echo lib/modules/${KVERSION}/modules.builtin >> /output/initrd-modules && \
    find lib/firmware -type f > /output/initrd-firmware && \
    find usr/lib/firmware -type f | sed 's!usr/!!' >> /output/initrd-firmware

# Copy output assets
RUN cd /usr/src/root && \
    cp -r usr/src/linux-headers* /output/headers && \
    cp -r lib/firmware /output/lib/firmware && \
    cp -r lib/modules /output/lib/modules && \
    cp boot/System.map* /output/System.map && \
    cp boot/config* /output/config && \
    cp boot/vmlinuz-* /output/vmlinuz && \
    echo ${KVERSION} > /output/version
