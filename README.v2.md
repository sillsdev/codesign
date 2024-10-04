# Codesign [![sign.yml reusable workflow.](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml)

The repo also contains the now deprecated front end reusable workflow `sign.yml`, and the actions and subworkflow that implement it along with tests/examples showing how to use them. These are effectively halting development at v2.1.

The simplest interface is the `sillsdev/codesign/.github/workflows/sign.yml` reusable workflow. This allows the signing of a multiple files, by calling into `sillsdev/codesign/.github/workflows/sign-digest.yml` to pass the detached digest for signing on our in-house code signing system. Files to be signed must be passed in an artifact, which will be overwritten with the signed versions when the `sign-digest.yml` workflow completes.

There are a several actions that facilitate more integrapted use of sign-digest.yml, by removing the need to pass files eclusively via artifacts. These break out the phases into windows only actions, permiting integration of the generate, apply, timestamp and verify phases into existing workflows where convenient. These are:

1. `sillsdev/codesign/generate-digest`: Generates a job to be passed to the `sign-digest.yml`, creating a temporary artifact to hold intermediate state.
2. `sillsdev/codesign/.github/workflows/sign-digest.yml`: Communicates with our remote signer, and provides a signed-digest job to be used in the signature attachment phase.
3. `sillsdev/codesign/apply-signed-digest`: Completes the signing process using the job from `workflow/sign-digest.yml`, by attaching the signed-digests to the files, deletes the temporay artifact, and then places the signed files into an optional directory (creating it as necessary), or into the workspace directory otherwise.
4. `sillsdev/codesign/timestamp`: Given correctly signed files attaches a secure timetamp, retying as necessary upto 10 times.
5. `sillsdev/codesign/verify-signature`: A simple way to test if files have a valid signature.

The generate-digest, timestamp, and verify-signature actions take a `path:` parameter that works the same way as `actions/upload-artifact`'s `path:` parameter, allowing you to specify files using a set of globs or filenames, one per line. e.g.:
```yaml
      uses: sillsdev/codesign/generate-digest@v2
      with:
        path: |
          gdlpp.exe
          grcompiler.exe
          *.dll
```

Please see the test files [sign-example.yml](https://github.com/sillsdev/codesign/blob/main/.github/workflows/sign-example.yml), and [actions-example.yml](https://github.com/sillsdev/codesign/blob/main/.github/workflows/actions-example.yml), for examples of how to use these resources.
