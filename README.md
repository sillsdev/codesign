# Codesign <br/> [![sign.yml reusable workflow.](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-example.yml) [![Demonstrate simple trusted-signing-action.](https://github.com/sillsdev/codesign/actions/workflows/sign-trusted-signing-example.yml/badge.svg)](https://github.com/sillsdev/codesign/actions/workflows/sign-trusted-signing-example.yml)

This repo provides a facade action over Microsoft's [azure/trusted-signing-action](https://github.com/Azure/trusted-signing-action). This Action takes care of setting required presets, and hides the many account and authentication parameters behind a secret JSON bundle. It also passes through all other parameters for selecting files, and specifying descriptions, and will select the right `certificate-profile-name` input based on a testing flag.

Please see further down for the documentation for the original [v2 documentation](/sillsdev/codesign/README.v2.md)


## Runner requirements

All the actions in the repo will only run on a windows runner, the GitHub hosted windows runner will work, as will self-hosted runners with Windows 7+ which have Powershell 5.1+, and .NET runtime 6.0+

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

This Action connects to the Azure service via OpenID Connect only, we do not support any of the other authentication mechanisms. All the account and authentication parameters are supplied via a secret JSON bundle defined like so, and base64 encoded:
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
Tells the signing service to use our Public Trust Test certificate, instead of the production certificate. This certificate is guarenteed not validate. Defaults to 'false'.


## `azure/trusted-signing-action` parameters
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
* `trace` except if the runner is debug mode, then it is always on.

### Unsupported
The following parameters from the Azure Action are not accepted as they are supplied by, or superceded by this Actions `credentials` parameter:
* `azure-tenant-id`
* `azure-client-id`
* `azure-client-secret`
* `azure-username`
* `azure-password`
* `endpoint`
* `trusted-signing-account-name`

The following Azure parameters preset by this Action, and so not accepted as inputs:
* `certificate-profile-name` derived from `use-test-certificate`
* `file-digest` set to SHA256.
* `timestamp-rfc3161` always uses http://timestamp.acs.microsoft.com.
* `timestamp-digest` set to SHA256.
* all `exclude-*-credential` inputs as we only support one authentication mechanism.
