FROM alpine:3.6

ENV GEARMAND_VERSION 1.1.16
ENV GEARMAND_SHA1 722b42b6d9a68dab24c83c71ff3fa1c2288b9c06

RUN addgroup -S gearman && adduser -G gearman -S -D -H -s /bin/false -g "Gearman Server" gearman

COPY patches/libhashkit-common.h.patch /libhashkit-common.h.patch
COPY patches/libtest-cmdline.cc.patch /libtest-cmdline.cc.patch

RUN set -x \
	&& apk add --no-cache --virtual .build-deps \
		wget \
		tar \
		ca-certificates \
		file \
		alpine-sdk \
		gperf \
		boost-dev \
		libevent-dev \
		util-linux-dev \
		hiredis-dev \
		libressl-dev \
	&& wget -O gearmand.tar.gz "https://github.com/gearman/gearmand/releases/download/$GEARMAND_VERSION/gearmand-$GEARMAND_VERSION.tar.gz" \
	&& echo "$GEARMAND_SHA1  gearmand.tar.gz" | sha1sum -c - \
	&& mkdir -p /usr/src/gearmand \
	&& tar -xzf gearmand.tar.gz -C /usr/src/gearmand --strip-components=1 \
	&& rm gearmand.tar.gz \
	&& cd /usr/src/gearmand \
	&& patch -p1 < /libhashkit-common.h.patch \
	&& patch -p1 < /libtest-cmdline.cc.patch \
	&& ./configure \
		--sysconfdir=/etc \
		--localstatedir=/var \
		--with-mysql=no \
		--with-postgresql=no \
		--disable-libpq \
		--disable-libtokyocabinet \
		--disable-libdrizzle \
		--disable-libmemcached \
		--enable-ssl \
		--enable-hiredis \
		--enable-jobserver=no \
	&& make \
	&& make install \
	&& cd / && rm -rf /usr/src/gearmand \
	&& rm /*.patch \
	&& runDeps="$( \
		scanelf --needed --nobanner --recursive /usr/local \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
	&& apk add --virtual .gearmand-rundeps $runDeps \
	&& apk del .build-deps \
	&& /usr/local/sbin/gearmand --version

COPY gearmand.conf /etc/gearmand.conf

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

USER gearman
EXPOSE 4730
CMD ["gearmand"]
