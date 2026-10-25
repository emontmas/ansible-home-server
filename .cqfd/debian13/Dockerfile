FROM debian:13

RUN apt update && \
    apt install -y \
        ansible \
        cargo \
        curl \
        git \
        rsync

ADD --checksum=sha256:d3a93702160e0ec03e2a4e996855db1f01adee801fb84a43add24e0877ef8eae \
    --chmod=0755 \
    https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh /tmp/
RUN /tmp/install-from-binstall-release.sh
RUN cargo binstall --install-path /usr/local/bin podlet

ADD ansible-requirements.yaml ansible-requirements.yaml
RUN ansible-galaxy install -r ansible-requirements.yaml -p /etc/ansible/roles
