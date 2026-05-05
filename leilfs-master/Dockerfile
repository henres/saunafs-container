ARG BASE_IMAGE=leilfs-base
FROM $BASE_IMAGE

# SAUNAFS_VERSION: pin a specific version (e.g. "3.14.0") or leave empty for latest
ARG SAUNAFS_VERSION=""

RUN apt-get update && \
	PKG="saunafs-master${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	apt-get install -y "$PKG" && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

# Define a non-volume-mapped path for templates
ENV LEILFS_TEMPLATE_DIR=/opt/saunafs/templates

RUN mkdir -p ${LEILFS_TEMPLATE_DIR} && \
	cp /var/lib/saunafs/metadata.sfs.empty ${LEILFS_TEMPLATE_DIR}/metadata.sfs.empty

COPY leilfs-master.start.sh /leilfs-master.start.sh
RUN chmod +x /leilfs-master.start.sh

EXPOSE 9419 9420 9421

CMD ["/leilfs-master.start.sh"]
