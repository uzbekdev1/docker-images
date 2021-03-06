# Copyright (c) 2020,2021 Oracle and/or its affiliates. All rights reserved.
#
# Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.
#
# ORACLE DOCKERFILES PROJECT
# --------------------------
# This is the Dockerfile for Oracle JDK 15
#
# REQUIRED FILES TO BUILD THIS IMAGE
# ----------------------------------
#
# (1) jdk-15.XX_linux-x64_bin.tar.gz
#     Download from https://www.oracle.com/java/technologies/javase-jdk15-downloads.html
#
# HOW TO BUILD THIS IMAGE
# -----------------------
# Put all downloaded files in the same directory as this Dockerfile
# Run:
#      $ docker build -t oracle/jdk:15 .
#
# This command is already scripted in build.sh so you can alternatively run
#		$ bash build.sh
#
# The builder image will be used to uncompress the tar.gz file with the Java Runtime.

FROM oraclelinux:7-slim as builder

MAINTAINER Aurelio Garcia-Ribeyro <aurelio.garciaribeyro@oracle.com>

RUN set -eux; \
	yum install -y \
		gzip \
		tar \
	; \
	rm -rf /var/cache/yum
	
# Default to UTF-8 file.encoding
ENV LANG en_US.UTF-8


# Environment variables for the builder image.  
# Required to validate that you are using the correct file

ENV JAVA_PKG=jdk-15.0.2_linux-x64_bin.tar.gz \
	JAVA_SHA256=54b29a3756671fcb4b6116931e03e86645632ec39361bc16ad1aaa67332c7c61 \
	JAVA_HOME=/usr/java/jdk-15

##
COPY $JAVA_PKG /tmp/jdk.tgz
RUN set -eux; \
	echo "$JAVA_SHA256 */tmp/jdk.tgz" | sha256sum -c -; \
	mkdir -p "$JAVA_HOME"; \
	tar --extract --file /tmp/jdk.tgz --directory "$JAVA_HOME" --strip-components 1; \
	rm /tmp/jdk.tgz; 
	
## Get a fresh version of SLIM for the final image	
FROM oraclelinux:7-slim

# Default to UTF-8 file.encoding
ENV LANG en_US.UTF-8

ENV JAVA_VERSION=15.0.2 \
	JAVA_HOME=/usr/java/jdk-15

ENV	PATH $JAVA_HOME/bin:$PATH	

# Copy the uncompressed Java Runtime from the builder image
COPY --from=builder $JAVA_HOME $JAVA_HOME

RUN set -eux; \
	yum install -y \
# JDK assumes freetype is available		
		freetype fontconfig \
	; \
	rm -rf /var/cache/yum; \
	ln -sfT "$JAVA_HOME" /usr/java/default; \
	ln -sfT "$JAVA_HOME" /usr/java/latest; \
	for bin in "$JAVA_HOME/bin/"*; do \
		base="$(basename "$bin")"; \
		[ ! -e "/usr/bin/$base" ]; \
		alternatives --install "/usr/bin/$base" "$base" "$bin" 20000; \
	done; \
# -Xshare:dump will create a CDS archive to improve startup in subsequent runs	
	java -Xshare:dump; \
	java --version; \
	javac --version; \
	grep VERSION= /etc/os-release

CMD ["jshell"]
