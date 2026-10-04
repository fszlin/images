# Agent instructions

## Keep secrets out of Git and container images

- Never write real credentials into source code, Dockerfiles, scripts, examples,
  documentation, commit messages, or build arguments. Use obvious placeholders,
  environment-variable references, or runtime-mounted secret files instead.
- Keep local credentials and generated state ignored. Examples include `.env`
  files, `.token`, `.git-credentials`, `auth.json`, private keys, cloud credential
  files, and `.opencode/`. Example environment files must contain placeholders only.
- Maintain `.dockerignore` in each image's build-context directory so credentials
  cannot enter the build context. The root `.dockerignore` does not cover builds
  whose contexts are `images/opencode` or `images/devops-agent`.

## Before committing or pushing

1. Review `git status --short` and the intended file list. Stage explicit paths
   after review; do not force-add ignored credential files.
2. Before committing, inspect `git diff --cached` and the full contents of newly
   added files for tokens, passwords, private keys, connection strings, and URLs
   containing credentials. Check scripts, configuration, logs, and generated files,
   not just files with secret-related names. Variable names and secret references
   are not themselves credentials.
3. Before pushing, identify the destination branch and inspect every outgoing
   commit, including intermediate versions of files. A secret deleted in a later
   commit is still exposed by earlier commits. For a new branch, include history
   that is not already on the destination remote. If the outgoing range cannot be
   established, resolve that before pushing.
4. If a suspected secret is found, stop the commit or push and report its file,
   location, and type without reproducing its value. Remove it from the proposed
   changes and review again. Do not rewrite existing history without the user's
   permission. If it has already been pushed, advise revoking or rotating it;
   deleting it from the latest file is insufficient.
5. Report what was reviewed and any limits. Do not claim that a repository is
   secret-free based only on ignore rules or a simple pattern search.

These instructions require agent review; they are not an automated Git hook or
server-side push protection.
