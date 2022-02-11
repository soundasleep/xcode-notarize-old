# Xcode Notarization

![notarize](https://user-images.githubusercontent.com/22433243/153662864-191f43f7-359f-41c9-b80d-88c617c5d2d6.png)

Github Action to notarize macOS applications or packages. It does this by submitting your built `.app`, `.pkg` or `.dmg` file to Apple's notarization service. It also will poll the notarization service until it times out of receives a success response.

_Note: This action is a fork from [devbotsxyz/xcode-notarize](https://github.com/devbotsxyz/xcode-notarize) with higher retries and timeout values._

* * *

## Basic Usage

```yaml
- name: "Notarize Release Build"
  uses: GuillaumeFalourd/xcode-notarize@main
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

## Full example

- [StackOverflow Reference](https://stackoverflow.com/questions/70991268/how-to-sign-and-notarize-a-pkg-within-a-github-actions-macos-runner)

Example of what you need to perform the full operation to generate, sign and notarize a PKG on Github Actions (the concept would be similar for APP or DMG).

### Premisses

- Email/Username Apple Id Developer account
- Password Apple Id Developer account
- Developer Id Installer: 
  - name
  - .cer base64 
  - .key base64 
  - .key password (if encrypted key, otherwise remove `-passin pass:${{ secrets.KEY_PASSWORD }}`)
- Developer Id Application:
  - name
  - .cer base64 
  - .key base64
  - .key password (if encrypted key, otherwise remove `-passin pass:${{ secrets.KEY_PASSWORD }}`)

### Workflow

```yaml
jobs:
  [...] # Job generating macos-bin-files from the project

  pkg:
    needs: [...]
    runs-on: macos-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Download macos bin file
        uses: actions/download-artifact@v2
        with:
          name: macos-bin-files
          path: dist/
      - name:
        run: | 
          ----- Create certificate files from secrets base64 -----
          echo ${{ secrets.DEVELOPER_ID_INSTALLER_CER_BASE64 }} | base64 --decode > certificate_installer.cer
          echo ${{ secrets.DEVELOPER_ID_INSTALLER_KEY_BASE64 }} | base64 --decode > certificate_installer.key
          echo ${{ secrets.DEVELOPER_ID_APPLICATION_CER_BASE64 }} | base64 --decode > certificate_application.cer
          echo ${{ secrets.DEVELOPER_ID_APPLICATION_KEY_BASE64 }} | base64 --decode > certificate_application.key

          ----- Create p12 file -----
          openssl pkcs12 -export -name zup -in certificate_installer.cer -inkey certificate_installer.key -passin pass:${{ secrets.KEY_PASSWORD }} -out certificate_installer.p12 -passout pass:${{ secrets.P12_PASSWORD }}
          openssl pkcs12 -export -name zup -in certificate_application.cer -inkey certificate_application.key -passin pass:${{ secrets.KEY_PASSWORD }} -out certificate_application.p12 -passout pass:${{ secrets.P12_PASSWORD }}

          ----- Configure Keychain -----
          KEYCHAIN_PATH=$RUNNER_TEMP/app-signing.keychain-db
          security create-keychain -p "${{ secrets.KEYCHAIN_PASSWORD }}" $KEYCHAIN_PATH
          security set-keychain-settings -lut 21600 $KEYCHAIN_PATH
          security unlock-keychain -p "${{ secrets.KEYCHAIN_PASSWORD }}" $KEYCHAIN_PATH

          ----- Import certificates on Keychain -----
          security import certificate_installer.p12 -P "${{ secrets.P12_PASSWORD }}" -A -t cert -f pkcs12 -k $KEYCHAIN_PATH
          security import certificate_application.p12 -P "${{ secrets.P12_PASSWORD }}" -A -t cert -f pkcs12 -k $KEYCHAIN_PATH
          security list-keychain -d user -s $KEYCHAIN_PATH

          ----- Codesign files with script -----
          use a script to sign each file from the artifact (ref: https://gist.github.com/GuillaumeFalourd/4efc73f1a6014b791c0ef223a023520a)

          ----- Generate PKG from codesigned files -----
          use a macos installer script (ref: https://github.com/KosalaHerath/macos-installer-builder/tree/master/macOS-x64)

          ----- Sign PKG file -----
          productsign --sign "${{ secrets.DEVELOPER_ID_INSTALLER_NAME }}" $INPUT_FILE_PATH $OUTPUT_FILE_PATH

          - name: "Notarize Release Build PKG"
            uses: devbotsxyz/xcode-notarize@v1 
            with:
              product-path: $PATH_TO_PKG
              appstore-connect-username: ${{ secrets.APPLE_ACCOUNT_USERNAME }}
              appstore-connect-password: ${{ secrets.APPLE_ACCOUNT_PASSWORD }}
              primary-bundle-id: 'BUNDLE_ID'

          - name: "Staple Release Build"
            uses: devbotsxyz/xcode-staple@v1
            with:
              product-path: $PATH_TO_PKG

  [...] # Job distributing or publising package

```

## License and Contributions

This Action is licensed under the [MIT](LICENSE) license. Contributions are very much welcome and encouraged but we would like to ask to file an issue before submitting pull requests. 
