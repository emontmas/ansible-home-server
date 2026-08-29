FROM debian:13

RUN apt update && \
    apt install -y \
        ansible \
        git

ADD ansible-requirements.yaml ansible-requirements.yaml
RUN ansible-galaxy install -r ansible-requirements.yaml
