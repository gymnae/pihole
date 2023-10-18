## pi-hole with automated list updates & unbound included

This image combines the work of [jacklul](https://github.com/jacklul/) and [chriscrowe](https://github.com/chriscrowe) to create a pi-hole solution which
1. Updates ad-blocker lists automatically and regularly
2. Ingrates unbound for acting as a [DNS resolver](https://docs.pi-hole.net/guides/dns/unbound/)
3. Runs as a self-contained docker image
4. Builds for arm64 & amd64

This approachs enables you to spin up one container and have a local or remote hosted pi-hole.

## Notes on remote hosting of pi-hole
Never expose port 53 over the Internet.
Instead, put a DNS-over-TLS or DNS-over-HTTPS service in front. If you are using a reverse proxy like caddy v2, it's quite straightforward to set up.

For DNS-over-TLS I use stunnel:
````
docker run -itd --name dot \
        --restart always \
        -e STUNNEL_SERVICE=dot \
        -e STUNNEL_ACCEPT=853 \
        -e STUNNEL_CONNECT=pi-hole:53 \
        -e STUNNEL_DEBUG=3 \
        -p 853:853 \
        -p 853:853/udp \
        --network=<yourdockernetwork> \
        -v <pathtocertificatestorage>/<yourdnsovertlsdomain>/<yourdnsovertlsdomain>.key:/etc/stunnel/stunnel.key:ro \
        -v </pathtocertificatestorage>/<yourdnsovertlsdomain>/<yourdnsovertlsdomain>.crt:/etc/stunnel/stunnel.pem:ro \
    dweomer/stunnel
````

For DNS-over-HTTPS I use satishweb doh-server:
````
docker run -itd --name doh-server \
    --hostname=doh-server \
    --network=<yourdockernetwork> \
    --name=doh-server \
    --restart unless-stopped \
    -e DOH_HTTP_PREFIX="/dns-query" \
    -e DOH_SERVER_LISTEN=":8053" \
    -e DOH_SERVER_TIMEOUT="10" \
    -e DOH_SERVER_TRIES="3" \
    -e UPSTREAM_DNS_SERVER=udp:pi-hole:53 \
satishweb/doh-server
````
