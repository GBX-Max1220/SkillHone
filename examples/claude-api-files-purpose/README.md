# Claude API Files upload — stale `purpose` parameter

This case reproduces an API example defect in Anthropic's public
[`claude-api` Skill](https://github.com/anthropics/skills/tree/main/skills/claude-api).
Before the upstream repair, multiple Managed Agents upload examples supplied a
`purpose` field to the Files API. The upstream fix states that the upload
endpoint does not accept this parameter and removes it from the examples.

The fixture is fetched at a pinned upstream commit by `prepare_fixture.py`.
It does not vendor the Skill source, execute upstream files, or require API
credentials. Its regression test is a deterministic static contract check: it
detects the unsupported multipart field and SDK argument in upload examples.
It does **not** call Anthropic's API or prove the server's response at runtime.

## Reproduce

From the SkillHone repository root:

```bash
python3 examples/claude-api-files-purpose/prepare_fixture.py /tmp/claude-api-files-purpose
python3 /tmp/claude-api-files-purpose/.test/test_files_upload_contract.py
```

The baseline test is expected to fail on four examples:

- `curl/managed-agents.md`
- `shared/managed-agents-client-patterns.md`
- `shared/managed-agents-environments.md`
- `typescript/managed-agents/README.md`

To simulate the repair, remove the unsupported field from each reported upload
example, then rerun the test. It should pass.

## Failure evidence and repair scope

The failure is confined to examples that upload files through
`client.beta.files.upload(...)` or `POST /v1/files`. A repair should remove the
unsupported upload argument while preserving unrelated request fields and
guidance. The test does not prescribe an exact patch or require a model call.

## Provenance and limitations

`PROVENANCE.md` in the generated fixture records the source commit, verified
Git tree and license object IDs, and the public upstream repair. The generated
fixture contains the complete pinned Skill tree so the issue can be encountered
through ordinary Skill use; only the regression test is scoped to this defect.

This case tests stale API guidance through a structural contract. It does not
validate every SDK version, server-side error wording, or live API behavior.
