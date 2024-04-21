원문: https://developer.apple.com/documentation/bundleresources/privacy_manifest_files
참고자료: https://green1229.tistory.com/434


> [!warning]
> You need to include a privacy manifest file in your third-party SDK if it’s listed in “SDKs that require a privacy manifest and signature,” in [Upcoming third-party SDK requirements](https://developer.apple.com/support/third-party-SDK-requirements/). Otherwise, include a privacy manifest file in your third-party SDK if it uses required reasons API, collects data about the person using apps that include the third-party SDK, enables the app to collect data about people using the app, or contacts tracking domains. Providing a privacy manifest file helps app developers to understand the API use and data-collection practices of your third-party SDK.
## Describing data use in privacy manifests
Declare the data collected by your app or by third-party SDKs.
## Describing use of required reason API
Ensure your use of covered API is consistent with policy.
