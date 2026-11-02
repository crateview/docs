Review the release notes, set `CRATEVIEW_VERSION` to the desired `v`-prefixed release tag in `.env`, and replace the frontend container:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

For example, the initial release uses `CRATEVIEW_VERSION={{RELEASE_TAG}}`. To roll back, restore the previous tag and repeat these commands. Keep the runtime environment and `registry-data` volume; do not run `docker compose down --volumes`. Source builds remain available through `compose.build.yaml`.

Dependabot is configured in the source repository to propose dependency updates; it does not upgrade your self-hosted deployment automatically. Re-test registry connectivity, sign-in and affected actions after an upgrade.
