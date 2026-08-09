# YourOwn.Chat RTCD release process

This repository supplies the RTCD source and image definition used by
YourOwn.Chat. It follows the same release separation as the patched
Mattermost server.

1. Reviewable changes land on `public-patched-1.2.6`.
2. The identical reviewed commit is promoted to `release-1.2-patched` for the
   dev-only preview pipeline. No production target is reachable from this
   branch.
3. A signed immutable patched-fork tag such as `v1.2.6-patched` is the only input eligible for a
   dev-to-production release. The deployment repository pins the full RTCD
   commit, builds it in Cloud Build, records SBOM/provenance, and deploys its
   immutable digest.

Do not deploy a floating branch, `latest` tag, or an upstream RTCD image. The
production image must be built from this repository's pinned commit and retain
the upstream license notice.
