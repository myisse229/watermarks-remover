# ChatGPT integration notes

This repository contains two Agent Skills, but ChatGPT does not install a skill by reading a GitHub repository or a SKILL.md file directly.

## Available components

- `skills/clean-user-facing-text/`: self-contained text-hygiene workflow. It can be used as a workflow reference, but its instruction file is not automatically loaded into ChatGPT.
- `skills/remove-ai-marks/`: thin HTTP client. It requires the Python service in `service/scripts/server.py` to be running at a URL reachable from the ChatGPT integration.

## Integration architecture

1. Deploy the Python HTTP service on a trusted HTTPS host (not a local loopback address).
2. Configure `WATERMARKS_SERVER_API_KEY` as a secret environment variable in the host. Never commit this key to Git.
3. Restrict access, enforce HTTPS, retain request-size limits, and avoid logging uploaded file contents.
4. Build a ChatGPT integration using a supported custom GPT Action or a remote MCP connector. Use the service's live `/openapi.json` as the API contract where supported.
5. Keep uploaded files private, obtain user authorization before modifying files, and return cleaned output as a new file rather than silently overwriting the original.

## Deployment notes

The service is standard-library Python and requires Python 3.10+. Its documented local command is:

```bash
python service/scripts/server.py --host 127.0.0.1 --port 8765
```

For a remote deployment, bind to the platform-provided port and a network interface reachable by the platform's reverse proxy. Confirm the CLI arguments and health endpoint against the checked-out server version before deploying. Do not expose the unauthenticated service publicly.

## Limits and responsible use

- Unicode and embedded metadata inspection/removal can be deterministic for supported formats, but some format features or external tools may be unavailable in a minimal host.
- Statistical watermark reduction is best-effort. It cannot prove human authorship, guarantee a detector result, or guarantee removal of secret-key marks.
- Preserve required provenance, legal, academic, and platform disclosures. Do not use the workflow to misrepresent authorship or falsify provenance.
- Metadata removal can remove legitimate provenance and invalidate digital signatures. Inspect first and preserve originals.

## Current state

Repository code and skill files are present. A ChatGPT account-level skill installation, public service deployment, and ChatGPT connector activation are separate operations and are not completed merely by committing this document.
