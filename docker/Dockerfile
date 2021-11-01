FROM nginx:1.21.3-alpine

# Copy binary to production image
COPY tailscale_bin/tailscaled /usr/sbin/tailscaled
COPY tailscale_bin/tailscale /usr/sbin/tailscale
RUN mkdir -p /var/run/tailscale /var/cache/tailscale /var/lib/tailscale

RUN apk add --update-cache \
        ca-certificates \
        iptables \
        ip6tables \
  && rm -rf /var/cache/apk/*

COPY ha_template.conf /etc/nginx/templates/ha.conf.template
COPY proxy_params.conf /etc/nginx/global/

COPY ./50-iniitalize-tailscale.sh /docker-entrypoint.d/
RUN chmod +x /docker-entrypoint.d/50-iniitalize-tailscale.sh

EXPOSE 8080

