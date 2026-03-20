ARG BASE_IMAGE=saunafs-base
FROM $BASE_IMAGE

# SAUNAFS_VERSION: pin a specific version (e.g. "3.14.0") or leave empty for latest
ARG SAUNAFS_VERSION=""

RUN apt-get update && \
	PKG="saunafs-master${SAUNAFS_VERSION:+=$SAUNAFS_VERSION}" && \
	apt-get install -y "$PKG" && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*

# Define a non-volume-mapped path for templates
ENV SAUNAFS_TEMPLATE_DIR=/opt/saunafs/templates

RUN mkdir -p ${SAUNAFS_TEMPLATE_DIR} && \
	cp /var/lib/saunafs/metadata.sfs.empty ${SAUNAFS_TEMPLATE_DIR}/metadata.sfs.empty

COPY saunafs-master.start.sh /saunafs-master.start.sh
RUN chmod +x /saunafs-master.start.sh

EXPOSE 9419 9420 9421

CMD ["/saunafs-master.start.sh"]
