ARG NODE_VERSION=12.18.3

FROM node:$NODE_VERSION-alpine as theia

ARG GITHUB_TOKEN
ARG version=latest

RUN apk add --no-cache make pkgconfig gcc g++ python libx11-dev libxkbfile-dev

WORKDIR /home/theia

ADD $version.package.json ./package.json
RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean

FROM node:$NODE_VERSION-alpine

COPY --from=theia /home/theia /home/theia

WORKDIR /home/theia

RUN adduser --disabled-password --gecos '' theia && \
    chmod g+rw /home && \
    mkdir -p /home/project && \
    chown -R theia:theia /home/theia && \
    chown -R theia:theia /home/project && \
    echo 'PS1="\[\033]0;$TITLEPREFIX:$PWD\007\]\n\[\033[32m\]\u@\h \[\033[33m\]\w\[\033[36m\]\[\033[0m\]\n$ "' >> /home/theia/.bashrc

RUN apk add --no-cache libsecret git bash curl gcc musl-dev

# install go
ENV GOLANG_VERSION 1.13.15

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		openssl \
		go \
	; \
	export \
# set GOROOT_BOOTSTRAP such that we can actually build Go
		GOROOT_BOOTSTRAP="$(go env GOROOT)" \
# ... and set "cross-building" related vars to the installed system's values so that we create a build targeting the proper arch
# (for example, if our build host is GOARCH=amd64, but our build env/image is GOARCH=386, our build needs GOARCH=386)
		GOOS="$(go env GOOS)" \
		GOARCH="$(go env GOARCH)" \
		GOHOSTOS="$(go env GOHOSTOS)" \
		GOHOSTARCH="$(go env GOHOSTARCH)" \
	; \
# also explicitly set GO386 and GOARM if appropriate
# https://github.com/docker-library/golang/issues/184
	apkArch="$(apk --print-arch)"; \
	case "$apkArch" in \
		armhf) export GOARM='6' ;; \
		armv7) export GOARM='7' ;; \
		x86) export GO386='387' ;; \
	esac; \
	\
	wget -O go.tgz "https://golang.org/dl/go$GOLANG_VERSION.src.tar.gz"; \
	echo '5fb43171046cf8784325e67913d55f88a683435071eef8e9da1aa8a1588fcf5d *go.tgz' | sha256sum -c -; \
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	\
	cd /usr/local/go/src; \
	./make.bash; \
	\
	rm -rf \
# https://github.com/golang/go/blob/0b30cf534a03618162d3015c8705dd2231e34703/src/cmd/dist/buildtool.go#L121-L125
		/usr/local/go/pkg/bootstrap \
# https://golang.org/cl/82095
# https://github.com/golang/build/blob/e3fe1605c30f6a3fd136b561569933312ede8782/cmd/release/releaselet.go#L56
		/usr/local/go/pkg/obj \
	; \
	apk del .build-deps; \
	\
	export PATH="/usr/local/go/bin:$PATH"; \
	go version

ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

USER theia

# Install VS Code Go tools
RUN GO111MODULE=on go get golang.org/x/tools/gopls@latest && \
    go get -u -v github.com/go-delve/delve/cmd/dlv && \
    go get -u -v github.com/ramya-rao-a/go-outline && \
    go get -u -v github.com/haya14busa/goplay/cmd/goplay && \
    go get -u -v github.com/fatih/gomodifytags && \
    go get -u -v github.com/josharian/impl && \
    go get -u -v golang.org/x/lint/golint && \
    go get -u -v honnef.co/go/tools/cmd/staticcheck && \
    go get -u -v github.com/cweill/gotests/... && \
    go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs

# Configure Theia
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins

EXPOSE 3000

ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
