# docker/Dockerfile

FROM ubuntu:20.04 as initrd_builder 

# Install necessary tools for building the initrd
RUN apt-get update && apt-get install -y \
    curl \
    cpio \
    gzip lz4 \
    linux-image-generic \
    initramfs-tools

# Copy the manifests, scripts, and initrd_root into the container
COPY initrd_root/ /app/initrd_root/

RUN cp /boot/vmlinuz-* /app/vmlinuz 

# Fetch the k3s binary
RUN mkdir -p app/initrd_root/usr/local/bin && \
    curl -L -o /app/initrd_root/usr/local/bin/k3s https://github.com/k3s-io/k3s/releases/download/v1.31.1%2Bk3s1/k3s && \
    chmod +x /app/initrd_root/usr/local/bin/k3s

# Unpack the initrd, customize it, and repack it
RUN mkdir -p /tmp/initrd-build && \
    cd /tmp/initrd-build && \
    file /boot/initrd.img-*  && \
    lz4 -dc /boot/initrd.img-* | cpio -idmv && \
    cp -r /app/initrd_root/* . && \
    find . | cpio --create --format='newc' |  lz4 -9  > /app/initrd-k3s.img

FROM ubuntu:20.04 
RUN apt-get update && apt-get install -y tftpd-hpa 
COPY --from=initrd_builder /app/initrd-k3s.img /var/lib/tftpboot/initrd-k3s.img
COPY --from=initrd_builder /app/vmlinuz /var/lib/tftpboot/vmlinuz
COPY tftp/pxelinux.cfg/ /var/lib/tftpboot/pxelinux.cfg/
EXPOSE 69-1024/udp
# Set the TFTP server to run on startup
ENTRYPOINT ["/usr/sbin/in.tftpd", "--foreground", "--listen", "--address", "0.0.0.0:69", "--secure", "/var/lib/tftpboot"]

