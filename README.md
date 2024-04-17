# Codesign [![sign.yml reusable workflow.](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml)

This contains a front end reusable workflow sign.yml, and the actions and subworkflow that implement it along with tests/examples showing how to use them.

The simplest interface is the `sillsdev/codesign/.github/workflows/sign.yml` reusable workflow. This allows the signing of a single file calling into `sillsdev/codesign/.github/workflows/sign-digest.yml` to pass the detach digest for signing on our in house code signing system. Files to be signed must be passed in as artifacts.

There are a several actions that facilitate direct use of sign-digest.yml, removing the need to pass files via artifacts, and permitting the potential to generate, apply, timestamp and verify multiple files in phases. These are:

1. `sillsdev/codesign/generate-digest`: Generates a digest to passed the `sign-digest.yml`, and an uniquely named artifact to be used in the signature attachement phase.
2. `sillsdev/codesign/.github/workflows/sign-digest.yml`: Communicates with our remote signer and provides a signed-digest to be used in the signature attachment phase.
3. `sillsdev/codesign/generate-digest`: Takes the outputs of the `sign-digest.yml` and `generate-digest` and applys the signed-digest to the file.
4. `sillsdev/codesign/timestamp`: Given a correctly signed files attaches a timetamp, retying as necessary upto 10 times.
5. `sillsdev/codesign/verify-signature`: A simple way to test if a file has a valid signature.

Please see the test files [sign-example.yml](https://github.com/sillsdev/codesign/blob/main/.github/workflows/sign-example.yml), and [actions-example.yml](https://github.com/sillsdev/codesign/blob/main/.github/workflows/actions-example.yml), for examples of how to use these resources.
