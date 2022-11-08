FROM alpine:latest AS build-base

RUN apk --quiet --no-cache add \
  build-base \
  zlib-dev \
  lua-dev \
  clang \
  cmake \
  make \
  mariadb-dev \
  && rm -rf /var/cache/apk/* \
  ;

#######################################################
FROM build-base as build

COPY . /src
WORKDIR /src

RUN cmake -G "Unix Makefiles" -H./ -B./build \
  -D WITH_LUA=true \
  -D WITH_MYSQL=true \
  -D CMAKE_INSTALL_PREFIX=/ \
  ../ && cd build && make

WORKDIR /src/build
RUN make install && chown -R 1001:1001 /var/pvpgn /etc/pvpgn

#######################################################
FROM alpine:latest AS runner

### Install dependencies
RUN apk --quiet --no-cache add \
  ca-certificates \
  libstdc++ \
  libgcc \
  libcurl \
  lua5.1-libs \
  mariadb-connector-c \
  && rm -rf /var/cache/apk/* \
  ;

### Copy build files
COPY --from=build \
  /sbin/bnetd \
  /sbin/bntrackd \
  /sbin/d2cs \
  /sbin/d2dbs \
  /sbin
COPY --from=build \
  /bin/bn* \
  /bin/sha1hash \
  /bin/tgainfo \
  /bin
COPY --from=build --chown=1001:1001 /etc/pvpgn /etc/pvpgn
COPY --from=build --chown=1001:1001 /var/pvpgn /var/pvpgn

### Prepare user
RUN addgroup --gid 1001 pvpgn \
  && adduser \
  --home /var/pvpgn \
  --gecos "" \
  --shell /sbin/nologin \
  --ingroup pvpgn \
  --system \
  --disabled-password \
  --no-create-home \
  --uid 1001 \
  pvpgn

### persist data and configs
VOLUME /var/pvpgn
VOLUME /etc/pvpgn

# expose served network ports
EXPOSE 6112

### Set user
USER 1001:1001

### RUN!
CMD ["bnetd", "-f"]
ENTRYPOINT ["bnetd"]