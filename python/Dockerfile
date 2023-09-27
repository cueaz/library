# syntax=docker/dockerfile:1.4
ARG PYTHON_VERSION=3.11
ARG DEBIAN_VERSION=12

FROM debian:$DEBIAN_VERSION AS builder

RUN set -eux; \
    # Setup flags
    # ref: https://gitlab.archlinux.org/archlinux/packaging/packages/pacman/-/blob/main/makepkg.conf
    arch="$(dpkg --print-architecture)"; \
    case "$arch" in \
        # ref: https://github.com/aws/aws-graviton-getting-started/blob/main/c-c++.md
        arm64) CFLAGS='-mcpu=neoverse-n1' ;; \
        # ref: https://github.com/clearlinux-pkgs/gcc/blob/main/gcc.spec
        amd64) CFLAGS='-march=westmere -mtune=sapphirerapids -fcf-protection' ;; \
        *) echo >&2 "error: unsupported architecture: $arch" && exit 1 ;; \
    esac; \
    # ref: https://bugs.gentoo.org/700012
    # also: dpkg-buildflags --get CFLAGS
    export CFLAGS="$CFLAGS -O3 -pipe -fno-plt -fexceptions \
                   -Wp,-D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security \
                   -fstack-protector-strong -fstack-clash-protection"; \
    export CXXFLAGS="$CFLAGS -Wp,-D_GLIBCXX_ASSERTIONS"; \
    export LDFLAGS='-Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -Wl,--strip-all'; \
    export MAKEOPTS="-j$(cat /proc/cpuinfo | grep processor | wc -l)"; \
    # Install dependencies
    # ref: https://github.com/python/cpython/blob/main/.github/workflows/posix-deps-apt.sh
    apt update; \
    apt install -y build-essential \
                   pkg-config \
                   git \
                   zlib1g-dev \
                   libbz2-dev \
                   liblzma-dev \
                   libffi-dev \
                   uuid-dev \
                   libssl-dev \
                   libb2-dev \
                   # not essential
                   libgdbm-dev \
                   libsqlite3-dev \
                   libncurses-dev \
                   libreadline-dev; \
    rm -rf /var/lib/apt/lists/*; \
    mkdir /work; \
    pushd /work; \
    # Build zlib (cloudflare fork)
    git clone --depth 1 https://github.com/cloudflare/zlib; \
    pushd zlib; \
    ./configure --static; \
    # switch '-mcpu=neoverse-n1' conflicts with '-march=armv8-a+crc' switch
    # ref: https://stackoverflow.com/questions/53138447/conflicts-between-cpu-and-arch-while-building
    sed -i 's/-march=armv8-a+crc//g' Makefile; \
    make "$MAKEOPTS" check; \
    popd; \
    # Build python
    git clone --depth 1 --branch "$PYTHON_VERSION" https://github.com/python/cpython; \
    pushd cpython; \
    # test modules as shared libraries
    sed -i '/^# Test modules/a \*shared\*' Modules/Setup.stdlib.in; \
    # fully ported to PY_STDLIB_MOD in 3.12
    # on 3.11, LIBFFI_LIBS, CURSES_LIBS, PANEL_LIBS, LIBREADLINE_LIBS are not yet available
    # ref: https://github.com/python/cpython/issues/90005
    sed -i 's/^#@MODULE__CTYPES_TRUE@\(.*\)/\1 @LIBFFI_LIBS@/' Modules/Setup.stdlib.in; \
    sed -i 's/^#@MODULE__CURSES_TRUE@\(.*\)/\1 @CURSES_LIBS@/' Modules/Setup.stdlib.in; \
    sed -i 's/^#@MODULE__CURSES_PANEL_TRUE@\(.*\)/\1 @PANEL_LIBS@/' Modules/Setup.stdlib.in; \
    sed -i 's/^#@MODULE_READLINE_TRUE@\(.*\)/\1 @LIBREADLINE_LIBS@/' Modules/Setup.stdlib.in; \
    cat Modules/Setup.stdlib.in; \
    ln -svrf Modules/Setup.stdlib Modules/Setup.local; \
    sed -i '/^ac_subst_vars=/a LIBFFI_LIBS\nCURSES_LIBS\nPANEL_LIBS\nLIBREADLINE_LIBS' configure; \
    ./configure --enable-option-checking=fatal \
                --enable-optimizations \
                --with-lto \
                --without-ensurepip \
                MODULE_BUILDTYPE=static \
                BZIP2_LIBS=-l:libbz2.a \
                # follow pkg-config deps by hand
                CURSES_LIBS='-l:libncursesw.a -l:libtinfo.a' \
                GDBM_LIBS=-l:libgdbm.a \
                LIBB2_LIBS=-l:libb2.a \
                LIBFFI_LIBS=-l:libffi.a \
                LIBLZMA_LIBS=-l:liblzma.a \
                LIBREADLINE_LIBS='-l:libreadline.a -l:libtinfo.a' \
                LIBSQLITE3_LIBS=-l:libsqlite3.a \
                LIBUUID_LIBS=-l:libuuid.a \
                PANEL_LIBS='-l:libpanelw.a -l:libncursesw.a -l:libtinfo.a' \
                ZLIB_LIBS=../zlib/libz.a \
                # do not link against libnsl (will be removed in 3.13)
                py_cv_module_nis=n/a; \
    make "$MAKEOPTS"; \
    make install DESTDIR=/rootfs; \
    popd; \
    popd; \
    # Strip out unnecessary files
    # ref: https://github.com/docker-library/python/blob/master/Dockerfile-linux.template
    set +x; before="$(set +x; find /rootfs)"; set -x; \
    find /rootfs/usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
         -o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
        \) -exec rm -rv '{}' +; \
    find /rootfs/usr/local -depth -type d -name __pycache__ -delete; \
    pyid="python$PYTHON_VERSION"; \
    pushd /rootfs/usr/local; \
    find . -maxdepth 1 ! -name bin ! -name lib ! -name . -exec rm -rv '{}' +; \
    # bin cleanup
    pushd bin; \
    find . -maxdepth 1 ! -name "$pyid" ! -name . -exec rm -rv '{}' +; \
    ln -sv "$pyid" python3; \
    ln -sv python3 python; \
    popd; \
    # lib cleanup
    pushd lib; \
    find . -maxdepth 1 ! -name "$pyid" ! -name . -exec rm -rv '{}' +; \
    pushd "$pyid"; \
    [ -z "$(ls lib-dynload | grep -v test | grep -v xxlimited)" ]; \
    # lib-dynload dir should exist to remove 'Could not find platform dependent libraries <exec_prefix>'
    rm -v lib-dynload/*; \
    # follow Debian convention
    rm -rv config-* \
           # python3-venv
           ensurepip \
           # python3-distutils (will be removed in 3.12)
           distutils \
           # python3-lib2to3
           lib2to3 \
           # idle-python3
           idlelib \
           # python3-tk
           tkinter \
           # python3-examples
           *demo; \
    popd; \
    popd; \
    popd; \
    # Print contents
    set +x; after="$(set +x; find /rootfs)"; set -x; \
    diff <(set +x; echo "$before") <(set +x; echo "$after") || true; \
    find /rootfs; \
    ldd -r /rootfs/usr/local/bin/python;

FROM gcr.io/distroless/cc-debian$DEBIAN_VERSION AS latest

COPY --link --from=builder /rootfs/ /
CMD ["python"]

FROM latest AS debug

COPY --link --from=busybox:musl /bin/ /bin/
CMD ["sh"]

FROM latest AS nonroot

USER nonroot
WORKDIR /home/nonroot

FROM debug AS debug-nonroot

USER nonroot
WORKDIR /home/nonroot

FROM latest AS check

RUN ["python", "--version"]
