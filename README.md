# Codesign <br/> [![sign.yml reusable workflow.](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml) [![Demonstrate simple trusted-signing-action.](https://github.com/sillsdev/codesign/actions/workflows/sign-trusted-signing-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-trusted-signing-example.yml)

This contains a facade action to Microsofts [azure/trusted-signing-action](https://github.com/Azure/trusted-signing-action), which takes care of setting required presets, and hides of the many account and authentication parameters in a secret JSON bundle. This action passes through all other parameters for selecting files to sign, and specifying descriptions. It reduces the `certificate-profile-name` input to a binary choice between our production and non-validating test certs.

Please see further down for the documentation for the original [v2 documentation](#Deprecated-v2-Actions-and-Workflow-documentation)


## Runner requirements

All the actions in the repo will only run on a windows runner, the GitHub hosted windows runner will work as will self-hosted runners with Windows 7+ which have Powershell 5.1+, and .NET runtime 6.0+

## Example

An example from an MSIX packaged app:
```yaml
- name: Sign with Trusted Signing
  uses: sillsdev/codesign/trusted-signing-action@v3
  with:
    credentials: ${{ secrets.TRUSTED_SIGNING_CREDENTIALS }}
    files-folder: ${{ github.workspace }}/artifacts/
    files-folder-filter: msixbundle,exe
    files-folder-recurse: true
    files-folder-depth: 4
    description:  'Release for version ${{ needs.build.outputs.version }} from branch ${{ github.ref_name }}'
    description-url: 'https://example.com'
```

## Usage

These are the specific inputs for the `sillsdev/codesign/trusted-signing-action`, unless indicate othewise you can use the same parameters as the underlying azure version.


### Authentication

This Action connects to the Azure service via OpenID Connect only we don't support any of the other authentication mechanisms. All the account and authentication parameters are supplied via a secret JSON bundle defined like so:
```json
{
  "tenant-id": "<Entra Directory ID>",
  "client-id": "<Entra Application ID>",
  "client-secret": "<Secret value, not secret ID>",
  "endpoint": "<end point the in the country your account is in>",
  "account-name": "<account name>"
}
```
We provide this bundle as an Organizational secret visible to selected repos called `TRUSTED_SIGNING_CREDENTIALS`.

### Test signing
```yaml
use-test-certificate: true
```
Don't use the Public Trust production certificate, instead use the Public Trust Test cert. This is guarenteed not validate. Defaults to 'false'.


## Azure action parameters
### Supported
We pass the following inputs through verbatim:
* `files`
* `files-folder`
* `files-folder-filter`
* `files-folder-recurse`
* `files-folder-depth`
* `files-catalog`
* `description`
* `description-url`
* `timeout`
* `batch-size`
* `trace` except if the runner is debug mode, then it is always on

### Unsupported
The following parameters from the Azure Action are not accepted as they are supplied by, or superceded by this Actions `credentials` parameter:
* `azure-tenant-id`
* `azure-client-id`
* `azure-client-secret`
* `azure-username`
* `azure-password`
* `endpoint`
* `trusted-signing-account-name`

The following Azure parameters predefined by this Action, and so not accepted:
* `certificate-profile-name` derived from `use-test-certificate`
* `file-digest` set to SHA256
* `timestamp-rfc3161` always uses http://timestamp.acs.microsoft.com
* `timestamp-digest` set to SHA256
* all `exclude-*-credential` inputs



# Deprecated v2 Actions and Workflow documentation
The repo also contains the now deprecated front end reusable workflow `sign.yml`, and the actions and subworkflow that implement it along with tests/examples showing how to use them. These are effectively halting development and v2.1.

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
