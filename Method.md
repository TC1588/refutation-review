# Method

## Step 1. Choose the dimensions

List the ways this system could fail without anyone noticing. Group them into four to six classes. One reviewer per class.

If two classes would read the same files for the same reason, merge them. If a class has no plausible quiet failure, drop it.

## Step 2. Reviewer prompt

> You are reviewing this codebase for one class of defect only: **[dimension]**.
>
> Read broadly before reporting. Ignore anything outside your class, including defects you are confident about. Another reviewer owns them.
>
> For each finding, return: the file and line, what the code does today, what it should do, and the concrete input or state that produces the wrong behavior. A finding with no failure scenario is not a finding.
>
> Do not report style, naming, or anything a linter would catch.

## Step 3. Verifier prompt, one per finding

> Below is a claimed defect. Your task is to refute it.
>
> [finding]
>
> Read the surrounding code and look for the reason this is not a bug. Guards elsewhere in the call path, a type constraint that makes the input impossible, a caller that already handles it, a misread of the control flow.
>
> Return REFUTED with the reason, or CONFIRMED with the failure scenario spelled out as concrete inputs leading to a concrete wrong output.
>
> Default to REFUTED when uncertain. A false positive costs more than a missed finding, because it spends trust.

## Step 4. Fan out without a barrier

Do not wait for all reviewers to finish before verifying. Each reviewer's findings spawn their verifiers as soon as that reviewer returns.

## Step 5. Verify before fixing

Confirmed is not the same as understood. Read each confirmed finding yourself before applying a fix. Never apply findings blindly.
