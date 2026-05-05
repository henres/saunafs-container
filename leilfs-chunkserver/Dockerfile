ARG BASE_IMAGE=leilfs-base
FROM $BASE_IMAGE

# SAUNAFS_VERSION: pin a specific version (e.g. "3.14.0") or leave empty for latest
ARG SAUNAFS_VERSION=""

RUN apt-get update && \
	PKG="saunafs-chunkserver${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	apt-get install -y "$PKG" && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

COPY leilfs-chunkserver.start.sh /leilfs-chunkserver.start.sh
RUN chmod +x /leilfs-chunkserver.start.sh

EXPOSE 9422

CMD ["/leilfs-chunkserver.start.sh"]
