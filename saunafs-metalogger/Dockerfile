ARG BASE_IMAGE=saunafs-base
FROM $BASE_IMAGE

# SAUNAFS_VERSION: pin a specific version (e.g. "3.14.0") or leave empty for latest
ARG SAUNAFS_VERSION=""

RUN apt-get update && \
	PKG="saunafs-metalogger${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	apt-get install -y "$PKG" && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

COPY saunafs-metalogger.start.sh saunafs-metalogger.start.sh
RUN chmod +x saunafs-metalogger.start.sh

CMD ["./saunafs-metalogger.start.sh"]
