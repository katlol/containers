# From https://github.com/containers/skopeo/blob/main/contrib/skopeoimage/stable/Containerfile

# stable/Containerfile
#
# Build a Skopeo container image from the latest
# stable version of Skopeo on the Fedoras Updates System.
# https://bodhi.fedoraproject.org/updates/?search=skopeo
# This image can be used to create a secured container
# that runs safely with privileges within the container.
#
FROM registry.fedoraproject.org/fedora:latest

# Don't include container-selinux and remove
# directories used by dnf that are just taking
# up space.
# TODO: rpm --setcaps... needed due to Fedora (base) image builds
#       being (maybe still?) affected by
#       https://bugzilla.redhat.com/show_bug.cgi?id=1995337#c3
RUN dnf -y update && \
    rpm --setcaps shadow-utils 2>/dev/null && \
    dnf -y install skopeo fuse-overlayfs \
        --exclude container-selinux && \
    dnf clean all && \
    rm -rf /var/cache /var/log/dnf* /var/log/yum.*

RUN useradd skopeo && \
    echo skopeo:100000:65536 > /etc/subuid && \
    echo skopeo:100000:65536 > /etc/subgid

# Copy & modify the defaults to provide reference if runtime changes needed.
# Changes here are required for running with fuse-overlay storage inside container.
RUN sed -e 's|^#mount_program|mount_program|g' \
        -e '/additionalimage.*/a "/var/lib/shared",' \
        -e 's|^mountopt[[:space:]]*=.*$|mountopt = "nodev,fsync=0"|g' \
        /usr/share/containers/storage.conf \
        > /etc/containers/storage.conf

# Setup the ability to use additional stores
# with this container image.
RUN mkdir -p /var/lib/shared/overlay-images \
             /var/lib/shared/overlay-layers && \
    touch /var/lib/shared/overlay-images/images.lock && \
    touch /var/lib/shared/overlay-layers/layers.lock

# Point to the Authorization file
ENV REGISTRY_AUTH_FILE=/tmp/auth.json

# Set the entrypoint
ENTRYPOINT ["/usr/bin/skopeo"]
