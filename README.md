# Xcode Notarize

![notarize](https://user-images.githubusercontent.com/22433243/153662864-191f43f7-359f-41c9-b80d-88c617c5d2d6.png)

Github Action to notarize macOS applications or packages. It does this by submitting your built `.app`, `.pkg` or `.dmg` file to Apple's notarization service. It also will poll the notarization service until it times out of receives a success response.

_Note: This action is a fork from [devbotsxyz/xcode-notarize](https://github.com/devbotsxyz/xcode-notarize) with higher retries and timeout values._

* * *

## Basic Usage

```yaml
- name: "Notarize Release Build"
  uses: devbotsxyz/xcode-notarize@v1
  with:
    product-path: "Export/Rings.app"
    appstore-connect-username: ${{ secrets.NOTARIZATION_USERNAME }}
    appstore-connect-password: ${{ secrets.NOTARIZATION_PASSWORD }}
```

Note that notarization is not the final step. After Apple has notarized your application, you also want to _staple_ a notarization ticket to your product. This can be done with the [Xcode Staple](https://github.com/marketplace/actions/xcode-staple) action.

* * *

## ▶️ Action Inputs

Field | Mandatory | Default Value | Observation
------------ | ------------  | ------------- | -------------
**product-path** | YES | N/A | Path to the product to notarize. <br/> _e.g: `path/to/product`_
**appstore-connect-username** | NO | N/A | The AppStore Connect username.
**appstore-connect-password** | NO | N/A | The AppStore Connect password.
**appstore-connect-api-key** | NO | N/A | The AppStore Connect API Key.
**appstore-connect-api-issuer** | NO | N/A | The AppStore Connect API Issuer.
**primary-bundle-id** | NO | N/A | Unique identifier that identifies this product notarization. Defaults to the bundle identifier of the app you are uploading.
**verbose** | NO | false | Verbose to detail action steps. 

## Related Actions

 * [Xcode Staple](https://github.com/marketplace/actions/xcode-staple) - Staple a Notarization Ticket to your product.

## License and Contributions

This Action is licensed under the [MIT](LICENSE) license. Contributions are very much welcome and encouraged but we would like to ask to file an issue before submitting pull requests. 
