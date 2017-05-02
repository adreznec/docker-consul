FROM ppc64le/alpine:edge
MAINTAINER James Phillips <james@hashicorp.com> (@slackpad)

# This is the release of Consul to pull in.
ENV CONSUL_VERSION=0.7.0

# Create a consul user and group first so the IDs get set the same way, even as
# the rest of this may change over time.
RUN addgroup consul && \
    adduser -S -G consul consul

ENV GOPATH $HOME/go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"

# Set up certificates, our base tools, and Consul.
RUN apk add --no-cache bash ca-certificates curl dumb-init git go gnupg g++ libcap make openssl wget && \
    mkdir -p /tmp/build && \
    cd /tmp/build && \
    wget -O gosu "https://github.com/tianon/gosu/releases/download/1.10/gosu-ppc64el" && \
    chmod +x gosu && \
    cp gosu /bin && \
    go get github.com/mitchellh/gox && \
    git clone --branch v${CONSUL_VERSION} --depth 1 https://github.com/hashicorp/consul.git $GOPATH/src/github.com/hashicorp/consul && \
    cd $GOPATH/src/github.com/hashicorp/consul && \
    make dev && \
    chmod +x bin/consul && \
    cp bin/consul /bin && \
    cd /tmp && \
    rm -rf /tmp/build && \
    apk del gnupg openssl && \
    rm -rf /root/.gnupg

# The /consul/data dir is used by Consul to store state. The agent will be started
# with /consul/config as the configuration directory so you can add additional
# config files in that location.
RUN mkdir -p /consul/data && \
    mkdir -p /consul/config && \
    chown -R consul:consul /consul

# Expose the consul data directory as a volume since there's mutable state in there.
VOLUME /consul/data

# Server RPC is used for communication between Consul clients and servers for internal
# request forwarding.
EXPOSE 8300

# Serf LAN and WAN (WAN is used only by Consul servers) are used for gossip between
# Consul agents. LAN is within the datacenter and WAN is between just the Consul
# servers in all datacenters.
EXPOSE 8301 8301/udp 8302 8302/udp

# CLI, HTTP, and DNS (both TCP and UDP) are the primary interfaces that applications
# use to interact with Consul.
EXPOSE 8400 8500 8600 8600/udp

# Consul doesn't need root privileges so we run it as the consul user from the
# entry point script. The entry point script also uses dumb-init as the top-level
# process to reap any zombie processes created by Consul sub-processes.
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

# By default you'll get an insecure single-node development server that stores
# everything in RAM, exposes a web UI and HTTP endpoints, and bootstraps itself.
# Don't use this configuration for production.
CMD ["agent", "-dev", "-client", "0.0.0.0"]
