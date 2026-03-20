ARG BASE_IMAGE=saunafs-base
FROM $BASE_IMAGE

# SAUNAFS_VERSION: pin a specific version (e.g. "3.14.0") or leave empty for latest
ARG SAUNAFS_VERSION=""

RUN apt-get update && \
	PKG_CLIENT="saunafs-client${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	PKG_ADM="saunafs-adm${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	apt-get install -y "$PKG_CLIENT" "$PKG_ADM" && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

COPY saunafs-client.start.sh saunafs-client.start.sh
RUN chmod +x saunafs-client.start.sh

ENTRYPOINT ["./saunafs-client.start.sh"]
