# Changesets

This folder tracks changes for versioned releases, the same way
[`mattpocock/skills`](https://github.com/mattpocock/skills) does.

## Workflow

When you add or change a skill:

```bash
npm install            # first time only (installs @changesets/cli)
npx changeset          # describe the change; pick major / minor / patch
```

That writes a markdown file here. To cut a release:

```bash
npx changeset version  # bumps package.json + rewrites CHANGELOG.md from pending changesets
git commit -am "release"
git push
gh release create v<new-version> --generate-notes
```

Consumers can then pin a version when installing via `npx skills add` or `/plugin install`.

See https://github.com/changesets/changesets for full docs.
