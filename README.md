# rtmpdump for Devario

Seafoam Labs hosts rtmpdump to provide a stable HTTPS source location for
building and distributing **Devario**. Devario's package infrastructure needs
reproducible access to dependencies even when the original upstream Git hosting
cannot be cloned.

This is a distribution-maintained source mirror of the work of the rtmpdump
contributors. Seafoam Labs does not claim upstream ownership and does not
represent this snapshot as a new upstream release.

## Why this mirror exists

The project itself is fine. Its Git hosting can no longer be cloned. Measured
2026-10-12, `git.ffmpeg.org` advertises `refs/heads/master` at the commit Arch
pins and answers `git ls-remote` normally, but it also advertises fifteen refs
whose value is the null OID:

```text
0000000000000000000000000000000000000000  refs/heads/tags/rel-1.9a
0000000000000000000000000000000000000000  refs/heads/tags/rel-1.9a@58
0000000000000000000000000000000000000000  refs/heads/tags/rel-2.0
... through refs/heads/tags/rel-2.3
```

Those are leftovers of the original SVN import that no longer resolve to an
object. A clone of a single branch ignores them and succeeds, but any client
that fetches every advertised ref, which is exactly what a source staging step
does, requests the null object and the server aborts it:

```text
fatal: remote error: upload-pack: not our ref 0000000000000000000000000000000000000000
```

Arch's own recipe still names that URL, and it built `1:2.6-2` on 2026-05-01,
so this is a regression on the hosting side rather than a packaging problem.
`git.ffmpeg.org` offers no tarball snapshot either; its gitweb snapshot URLs
return `text/html`. Four GitHub mirrors that advertise
`git.ffmpeg.org/rtmpdump` (`mirror/rtmpdump`, `Distrotech/rtmpdump`,
`mstorsjo/rtmpdump` and `lu-zero/rtmpdump`) were cloned and checked for the
object, and none of them contains the commit Arch pins, so none of them is a
usable substitute.

## Current source snapshot

The `upstream` branch carries the commit the Devario recipe pins:

```text
138fdb258d9fc26f1843fd1b891180416c9dc575
```

- Upstream commit date: **14 November 2025**.
- Upstream commit subject: **Version bump**.
- Package version: **1:2.6**, the commit the upstream annotated tag `v2.6`
  points at.
- The branch holds all 522 commits of upstream `master` history.
- The `v2.6` tag is upstream's own annotated tag object
  (`ac15ca4886752fc1937c7f2b45398d573edf2af8`, tagged by Howard Chu, unsigned);
  it is preserved byte for byte, not recreated here.

The history was fetched directly from `https://git.ffmpeg.org/rtmpdump` with an
explicit refspec, `+refs/heads/master:refs/heads/upstream` and
`+refs/tags/v2.6:refs/tags/v2.6`, which is what avoids the broken refs. Nothing
in this branch was rewritten: because a commit ID is a hash of its contents,
`138fdb258d...` here is proof that the snapshot is the same source Arch built.
The `1.x` branch and the null-OID refs are not carried.

## No snapshot tag on the pinned commit

This repository deliberately adds no tag of its own. Upstream's `v2.6` already
sits exactly on the pinned commit, so the recipe's

```sh
git describe --tags | sed 's/^v//'
```

returns `v2.6` and the version stays `2.6`. An added snapshot tag on the same
commit would be an equally valid match for `git describe`, and which one wins
depends on tag ordering, so a future version could silently drift. Keep it that
way if you add tags here.

## Building a Devario package

The root [PKGBUILD](PKGBUILD) builds **1:2.6-2** for x86_64. It clones this
repository over HTTPS and checks out the original upstream commit:

```text
seafoam-rtmpdump::git+https://github.com/Seafoam-Labs/rtmpdump.git#commit=138fdb258d9fc26f1843fd1b891180416c9dc575
```

The source is pinned by full commit ID and keeps the checksum the upstream
recipe declares, because `git archive` of that commit still hashes to
`0cba5d49d41b5e35a34ea8230124ed853eeeac19091d17a4188a47c31a17d352`. The source
cache and checkout are named `seafoam-rtmpdump` so a stale `rtmpdump` cache
pointing at `git.ffmpeg.org` is never reused.

Use an Arch-compatible `base-devel` environment with `gnutls` and `zlib`
installed, then:

```sh
makepkg
```

rtmpdump builds against either OpenSSL or GnuTLS; the recipe selects GnuTLS with
a `sed` over `Makefile` and `librtmp/Makefile`, exactly as the upstream recipe
does. [nettle-4.patch](nettle-4.patch) adapts the GnuTLS handshake path to the
nettle 4.0 API, which changes the arity of `hmac_sha256_digest`, and is applied
in `prepare()`.

## License and attribution

rtmpdump is copyright its upstream contributors and is distributed under the
**GNU General Public License, version 2.0 only** for the tools and the **GNU
Lesser General Public License, version 2.1 only** for `librtmp`, as the upstream
[COPYING](COPYING) file records. The original source copyright notices, the
documentation and the `librtmp` sources are preserved unmodified. Seafoam Labs'
role here is to host the source for Devario.
