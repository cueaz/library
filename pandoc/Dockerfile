FROM registry.gitlab.com/islandoftex/images/texlive:latest-full

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends pandoc; \
    rm -rf /var/lib/apt/lists/*; \
    pandocdir='/.pandoc'; \
    templatedir="$pandocdir/templates"; \
    install -d "$templatedir"; \
    ln -s "$pandocdir" /root; \
    repo='Wandmalfarbe/pandoc-latex-template'; \
    # ver="$(basename "$(curl -Ls -o /dev/null -w '%{url_effective}' "https://github.com/$repo/releases/latest")")"; \
    # for pandoc 3.1.3 in debian testing
    ver="v2.4.0"; \
    name='eisvogel'; \
    curl -L "https://raw.githubusercontent.com/$repo/$ver/$name.tex" -o "$templatedir/$name.latex"; \
    cachedir="$(find /usr/local/texlive -mindepth 2 -maxdepth 2 -type d -name texmf-var -print -quit)"; \
    chmod -R o+w "$cachedir";

WORKDIR /data
ENTRYPOINT ["/usr/bin/pandoc"]
