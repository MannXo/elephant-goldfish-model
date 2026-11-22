---
name: egm-drift-report
license: Apache-2.0
disable-model-invocation: true
description: >-
  Runs drift detection on all design documents to check for document and
  code consistency. Produces an actionable report with differences. Use when the
  user asks to detect drift or check for consistency across all design documents
  or READMEs in a directory.
---

# Detect Drift In Designs and Code

When running in Codex or ChatGPT Work, first read [Codex integration](../../docs/codex.md).
In Claude Code or Cowork, read `${CLAUDE_PLUGIN_ROOT}/docs/claude.md` first.

When the user asks to detect drift across all designs or a set of READMEs, the
goal is to identify all instances where the codebase has diverged or failed to
implement the specifications described by those designs or READMEs.

1.  **Check Input**: Verify a directory containing design documents or READMEs
    is provided. If not provided, ask the user to specify the target directory.
2.  **List Documents**: List all files in the design directory.
3.  **Analyze**: For EACH design document:

    *   **Contextualize**: Read the design document. Identify the intent,
        proposed changes, and specific filenames/symbols mentioned.
    *   **Locate Source**: Find the corresponding source code files in the
        source tree. If paths aren't explicit, use code search or grep to
        locate the relevant classes, functions, or API endpoints mentioned in
        the doc. Prompt the user to add missing links to the document to make
        future detection cheaper.
    *   **Verify**: Read the current state of those source files.
    *   **Detect Drift**: Compare the design vs. reality. Look specifically for:
        *   Missing Features: Logic described in the doc but absent in the code.
        *   Incomplete Models: Dataclasses/Classes/Structs/Protos missing fields
            specified in the design.
        *   Logic Shortcuts: Hardcoded values where dynamic logic was planned.
        *   Architectural Divergence: Implementation using different
            patterns/libraries than proposed.
    *   **Filter**: Report ONLY the differences. Do not describe what is working
        or what matches. If the code matches the design perfectly, output "No
        drift detected."
    *   **Cite Evidence**: For each difference, record the design requirement
        (doc path and line) and the code evidence (file and line, or the
        searches that found nothing). Do not report a difference without both.
    *   **Classify**: Mark each difference as one of:
        *   `Missing implementation`: the design asks for it and the code has
            no trace of it.
        *   `Intentional deviation`: the code differs on purpose. Cite the
            commit, issue, code comment or later design that records the
            choice.
        *   `Unexplained deviation`: the code differs and nothing records why.
    *   **Prioritize**: Rate each difference `High`, `Medium` or `Low` and give
        the reason in one sentence. Rate higher when a reader who trusts the
        design would build something broken, and lower when only wording or
        naming differs.
    *   **Propose**: Recommend one action: `Code change` (implement the
        design), `Design correction` (update the doc to match the code), or
        `Accept deviation` (a maintainer records why the code differs). An
        intentional deviation never gets a code change.

4.  **Report**: Produce a single report listing all discrepancies found.

    *   Append the analysis for each file to `{YYYY-MM-DD}_DRIFT_REPORT.md`
        using your file-writing tool.
    *   List findings in priority order, `High` first.
    *   Strict Format:

        ```markdown
        ## <DESIGN_DOC_NAME>

        *   **[<Priority>] <Classification>**: <divergence>
            *   Design: <doc path:line and the requirement>
            *   Code: <file:line, or the searches that found nothing>
            *   Priority: <one-sentence reason>
            *   Proposed action: <Code change | Design correction | Accept deviation>
        <Or "No drift detected">
        ```

    *   Do not edit code or design documents. The report proposes changes and
        a human approves them.

5.  **Guidance**

    -   Be rigorous. If a design doc mentions a specific object, field or file
        name and the code doesn't have it, that is a reportable drift. Double
        check for coverage. If there is some part of the logic that is not
        referenced in a design or README, this is a reportable drift.
    -   Read files liberally to ensure you are looking at the current state
        of the code, not relying on memory.
    -   Process the files sequentially to ensure the tool output buffer doesn't
        overflow.
