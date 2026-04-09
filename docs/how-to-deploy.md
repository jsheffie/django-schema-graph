# How to Deploy

This fork is not published to PyPI. It is installed directly from a GitHub release tarball — no git client required on the target machine.

## Creating a new release

**1. Bump the version** in `pyproject.toml`:

```toml
[tool.poetry]
version = "3.x.x"
```

**2. Update `CHANGELOG.md`** with the new version and release date under `## [Unreleased]`.

**3. Build the frontend bundle** (the compiled `main.js` must be committed):

```bash
yarn install --frozen-lockfile
yarn run build
```

**4. Commit, tag, and push:**

```bash
git add pyproject.toml CHANGELOG.md schema_graph/static/schema_graph/main.js
git commit -m "Bump version to 3.x.x"
git tag v3.x.x
git push && git push origin v3.x.x
```

**5. Create the GitHub release** from that tag:

```bash
gh release create v3.x.x --title "v3.x.x" --notes "Brief summary of changes"
```

---

## Installing in another project

Once the release exists, install via the tarball URL — no git required:

```
pip install https://github.com/jsheffie/django-schema-graph/archive/refs/tags/v3.x.x.tar.gz
```

### `requirements.txt`

```
https://github.com/jsheffie/django-schema-graph/archive/refs/tags/v3.x.x.tar.gz
```

### `pyproject.toml` (poetry or PEP 621)

```toml
django-schema-graph = {url = "https://github.com/jsheffie/django-schema-graph/archive/refs/tags/v3.x.x.tar.gz"}
```

### Dockerfile

```dockerfile
RUN pip install --no-cache-dir https://github.com/jsheffie/django-schema-graph/archive/refs/tags/v3.x.x.tar.gz
```

---

## Upgrading

1. Follow the release steps above to create the new tag and release.
2. In each consuming repo, update the tarball URL to the new tag version.
