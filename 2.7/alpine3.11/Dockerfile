FROM alpine:3.11
LABEL maintainer "Jamie Hewland <jhewland@gmail.com>"

# ensure local pypy is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

ENV PYPY_VERSION 7.3.1
ENV PYPY_SHA256SUM 3af22471c9b607a27db4c112064cddda78c3e2751dd6db7f8fbae75628d5ad90

RUN set -ex; \
    apk add --no-cache --virtual .fetch-deps \
        tar \
        xz \
    ; \
    \
    filename="pypy2.7-v${PYPY_VERSION}-linux64-alpine$(cut -d '.' -f 1,2 /etc/alpine-release).tar.bz2"; \
    wget -O pypy.tar.bz2 "https://github.com/JayH5/alpine-pypy/releases/download/2.7-$PYPY_VERSION/$filename"; \
    \
    echo "$PYPY_SHA256SUM *pypy.tar.bz2" | sha256sum -c -; \
    tar -xjC /usr/local --strip-components=1 -f pypy.tar.bz2; \
    find /usr/local/lib-python -depth -type d -a \( -name test -o -name tests \) -exec rm -rf '{}' +; \
    rm pypy.tar.bz2; \
    apk del .fetch-deps; \
    \
    apk add --no-cache findutils; \
    find /usr/local -type f -executable -not \( -path '*tkinter*' \) -exec scanelf --needed --nobanner --format '%n#p' '{}' ';' \
        | tr ',' '\n' \
        | sort -u \
        | awk 'system("[ -e /usr/local/lib/" $1 " ] || [ -e /usr/local/bin/" $1 " ]") == 0 { next } { print "so:" $1 }' \
        | xargs -rt apk add --virtual .pypy-rundeps; \
    apk del findutils; \
    \
    pypy --version

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 20.0.2

RUN set -ex; \
    \
    wget -O get-pip.py 'https://bootstrap.pypa.io/get-pip.py'; \
    \
    pypy get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        "pip==$PYTHON_PIP_VERSION" \
    ; \
    pip --version; \
    \
    rm -f get-pip.py

CMD ["pypy"]
