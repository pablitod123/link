# Link iOS

Welcome to the Plaid Link iOS!

In this repository you find instructions and sample applications in
[Objective-C](LinkDemo-ObjC), [Swift4](LinkDemo-Swift4), [Swift3](LinkDemo-Swift3) and [Swift 2](LinkDemo-Swift2) (requires Xcode 7)
that demonstrate Plaid Link for iOS. At the center of it all lies [`LinkKit.framework`][linkkit]
an embeddeable framework managing the details of linking an account with Plaid.

Here are some screenshots of the user interface provided by Plaid Link iOS:

![Examples of Plaid Link iOS](docs/images/link-ios-citi.jpg)

## Table of Contents

* [link-ios](#link-ios)
  * [Preparation](#preparation)
  * [Requirements](#requirements)
  * [Quick Guide for The Impatient (TL;DR)](#quick-guide-for-the-impatient)
  * [Getting Started](#getting-started)
    * [Integration](#integration)
      * [Cocoapods](#cocoapods)
      * [Carthage](#carthage)
      * [Manual](#manual)
    * [Configuration](#configuration)
    * [Implementation](#implementation)
      * [Objective-C](#objective-c)
      * [Swift](#swift)
  * [Custom Initializers](#user-content-custom-initializer-info)
    ([Objective-C](#user-content-objc-custom-initializer), [Swift](#user-content-swift-custom-initializer))
  * [Update Mode](#user-content-update-mode-info)
    ([Objective-C](#user-content-objc-update-mode), [Swift](#user-content-swift-update-mode))
  * [Customization](#customization)
  * [Troubleshooting](#troubleshooting)
  * [Known Issues](#known-issues)

## Preparation

You will need the following to integrate Plaid Link iOS:

* Xcode 7 or greater
* A Plaid `public_key` (available from the [Plaid Dashboard][dashboard-keys]).
* The latest version of the [`LinkKit.framework`][linkkit]

## Requirements

* iOS 8.0 or greater, since [`LinkKit.framework`][linkkit] is an embeddable framework

## Quick Guide For the Impatient

1. Embed the [`LinkKit.framework`][linkkit] into your application
2. Add a custom **Run Script Build Phase** that runs *after* the **Embed Frameworks Build Phase**
   and contains
   [this script](#user-content-prepare-distribution-script), to strip any non-ARM architectures
   from the embedded framework from distribution builds.
3. Import LinkKit
   ([Objective-C](#user-content-objc-import), [Swift](#user-content-swift-import))
4. Setup Plaid Link
   ([Objective-C](#user-content-objc-setup-shared), [Swift](#user-content-swift-setup-shared))
   Optional, but highly recommended! [Read why...](#setup-plaid-link)
5. Adopt the [`PLKPlaidLinkViewDelegate`](LinkKit.framework/Headers/PLKPlaidLinkViewController.h) protocol
   ([Objective-C](#user-content-objc-protocol), [Swift](#user-content-swift-protocol))
6. Implement the delegate methods
   ([Objective-C](#user-content-objc-delegate), [Swift](#user-content-swift-delegate))
7. Present the `PLKPlaidLinkViewController`
   ([Objective-C](#user-content-objc-present-custom), [Swift](#user-content-swift-present-custom))

## Getting Started

This repository contains one Xcode workspace named `LinkDemo.xcworkspace` with two sample projects,
one for Objective-C (`LinkDemo-ObjC.xcodeproj`) and one for Swift (`LinkDemo-Swift.xcodeproj`).
Both projects work very similar to illustrate the integration and use of Plaid Link iOS:

After the LinkDemo application has finished launching it will [setup Plaid Link iOS](#setup-plaid-link)
and once setup has been successful the application will enable a UIButton, which when tapped presents
the Plaid Link iOS user interface. When Plaid Link iOS is finished the application displays
the result with a `UIAlertViewController` and logs the result to the console using `NSLog`.

### Integration

#### Cocoapods

To integrate [`LinkKit.framework`][linkkit] into your application using
[CocoaPods](https://cocoapods.org/#install):

* Add the next line to your Podfile:

```
pod 'Plaid'
```

* Run the following command:
`pod install`

* Add a **Run Script Build Phase** (we recommend naming it `Prepare for Distribution`) with the
  [script below](#user-content-prepare-distribution-script).
  Be sure that the `Prepare for Distribution` build phase runs _after_ the `Embed
  Frameworks` (or `[CP] Embed Pods Frameworks` when using Cocoapods) build phase.
  <br><img src='docs/images/setup/07-new_run_build_phase.jpg' width='350' title='Add new Run Script Build Phase'>
  <img src='docs/images/setup/08-edit_run_build_phase.jpg' width='350' title='Editing the `Prepare for Distribution` build phase'>

* To update [`LinkKit.framework`][linkkit] in the future run:
`pod update Plaid`

#### Carthage

To integrate [`LinkKit.framework`][linkkit] into your application using
[Carthage](https://github.com/Carthage/Carthage):

* Add the next line to your Cartfile:

```
binary "https://raw.githubusercontent.com/plaid/link/master/LinkKit.json"
```

* Run the following command: `carthage update`

* This downloads the latest version of [`LinkKit.framework`][linkkit]
to `Carthage/Build/iOS/LinkKit.framework`

* Follow the [manual steps below](#manual)

#### Manual

* Get the latest version of [`LinkKit.framework`][linkkit]
* Embed the [`LinkKit.framework`][linkkit] into your application
  <br><img src='docs/images/setup/01-drag.jpg' width='350' title='Embedding LinkKit.framework'>
* Depending on the location of the [`LinkKit.framework`][linkkit] on the filesystem
  you may need to change the **Framework Search Paths** build setting to avoid the
  `fatal error: 'LinkKit/LinkKit.h' file not found`.
  The LinkDemo Xcode projects have it set to `FRAMEWORK_SEARCH_PATHS = $(PROJECT_DIR)/../`.
  <br><img src='docs/images/setup/05-edit_framework_search_path.jpg' width='350' title='Editing the `FRAMEWORK_SEARCH_PATHS` build setting'>
* Add a **Run Script Build Phase** (we recommend naming it `Prepare for Distribution`) with the
  [script below](#user-content-prepare-distribution-script).
  Be sure that the `Prepare for Distribution` build phase runs _after_ the `Embed
  Frameworks` build phase.
  <br><img src='docs/images/setup/07-new_run_build_phase.jpg' width='350' title='Add new Run Script Build Phase'>
  <img src='docs/images/setup/08-edit_run_build_phase.jpg' width='350' title='Editing the `Prepare for Distribution` build phase'>

<a name='prepare-distribution-script'></a>

The scripts removes code from the framework, which is included to support the iPhone Simulator,
but which may not be distributed via the App Store.

While the example below supports manual integration as well as integration via Cocoapods or
Carthage, be sure to change the path to `${LINK_ROOT:-$PROJECT_DIR}"/LinkKit.framework/prepare_for_distribution.sh`
according to your setup and quote the filepaths in case they contain whitespace.

```sh
LINK_ROOT=${PODS_ROOT:+$PODS_ROOT/Plaid/ios}
cp "${LINK_ROOT:-$PROJECT_DIR}"/LinkKit.framework/prepare_for_distribution.sh "${CODESIGNING_FOLDER_PATH}"/Frameworks/LinkKit.framework/prepare_for_distribution.sh
"${CODESIGNING_FOLDER_PATH}"/Frameworks/LinkKit.framework/prepare_for_distribution.sh
```

### Configuration

There are two ways that Plaid Link iOS can be configured:

1. either in code (see below for [Objective-C](#user-content-objc-setup-custom) and
   [Swift](#user-content-swift-setup-custom))
2.  or by adding a Plaid Link specific entry named `PLKPlaidLinkConfiguration` containing
    the following configuration items to the `Info.plist` of the application:

#### Required `PLKPlaidLinkConfiguration` items:

| Key          | Type            | Values                                              | Description                                                                                                                                                                                                                                                                                 |
| ---          | ---             | :---:                                               | ---                                                                                                                                                                                                                                                                                         |
| `clientName` | String          | —                                                   | Displayed to the user once they have successfully linked their account.                                                                                                                                                                                                                     |
| `key`        | String          | —                                                   | Your Plaid `public_key` available from the [Plaid dashboard][dashboard-keys]                                                                                                                                                                                                                |
| `env`        | String          | `Production`, `Tartan`¹, `Sandbox`², `Development`² | Select the environment to use. For development use `Development`; for testing, use `Sandbox` (unless you specifically require `APIv1`, in which case, use `Tartan`); and for release builds, choose `Production`. Depending on the `env`, LinkKit will talk to different Plaid API servers. |
| `product`    | String or Array | `auth`, `transactions`, `income`, or `identity`     | Select the Plaid products you would like to use (visit the [Plaid Products page](https://plaid.com/products/) to learn more)                                                                                                                                                                |

¹ For use with APIv1 only

² For use with APIv2 only

#### Optional `PLKPlaidLinkConfiguration` items:

| Key                                                 | Type       | Values¹              | Description                                                                                                                                                                                                                                                |
| ---                                                 | ---        | :---:                | ---                                                                                                                                                                                                                                                        |
| `webhook`                                           | String     | —                    | The webhook (a URL string starting with `http://` or `https://`) will receive notifications once a user's transactions have been processed and are ready for use. For details refer to the [Plaid API documentation](https://plaid.com/docs/api/#webhook). |
| <a name='config-select-account'>`selectAccount`</a> | Boolean    | `YES`, **`NO`**      | Whether the user should select a specific account after successfully linking their bank account                                                                                                                                                            |
| `apiVersion`                                        | String     | `APIv1`, **`APIv2`** | The Plaid API version to use, please specify `APIv1` only if you have an important reason to do so and have been enabled for `APIv1` use by Plaid.                                                                                                         |
| `customization`                                     | Dictionary | —                    | Link copy customization ([see below](#shared-configuration-customization))                                                                                                                                                                                 |

¹ _Default values are shown in_ **bold**.

#### Environment \ API Version Compatibility

| ↓`apiVersion` \ `env`→ | `Production` | `Tartan` | `Sandbox` | `Development` |
| ---                    | :---:        | :---:    | :---:     | :---:         |
| `APIv1`                | ✓            | ✓        | —         | —             |
| `APIv2`                | ✓            | —        | ✓         | ✓             |

Commandline aficionados may find the following command useful:

```sh
/usr/libexec/PlistBuddy "${PATH_TO_THE_APPLICATIONS}"/Info.plist \
  -c 'Add PLKPlaidLinkConfiguration:clientName string $(PRODUCT_NAME)' \
  -c 'Add PLKPlaidLinkConfiguration:key string $(LINK_KEY)' \
  -c 'Add PLKPlaidLinkConfiguration:env string $(LINK_ENV)' \
  -c 'Add PLKPlaidLinkConfiguration:product string auth' \
  -c 'Add PLKPlaidLinkConfiguration:selectAccount bool NO'
```

The command adds the necessary keys and values to the `Info.plist`. In this example the values
for `clientName`, `key`, and `env` will be set to the value of the Xcode build settings,
`PRODUCT_NAME` usually expands to the application name, the custom build settings `LINK_KEY`
and `LINK_ENV` need to be added manually in Xcode. This allows use of a different `key`
and `env` for the different build configurations (e.g. Debug, Release).

<img src='docs/images/setup/00a-edit_info_plist.jpg' width='350' title='Editing the Plaid Link configuration in Info.plist'>
<img src='docs/images/setup/11b-add_build_setting.jpg' width='350' title='Adding build settings'>

### Implementation 

* Import LinkKit
  ([Objective-C](#user-content-objc-import), [Swift](#user-content-swift-import))
* Setup Plaid Link
  ([Objective-C](#user-content-objc-setup-shared), [Swift](#user-content-swift-setup-shared))
  Optional, but highly recommended! [Read why...](#setup-plaid-link)
* Adopt the [`PLKPlaidLinkViewDelegate`](LinkKit.framework/Headers/PLKPlaidLinkViewController.h) protocol 
  ([Objective-C](#user-content-objc-protocol), [Swift](#user-content-swift-protocol))
* Implement the delegate methods
  ([Objective-C](#user-content-objc-delegate), [Swift](#user-content-swift-delegate))
* Present the `PLKPlaidLinkViewController`
  ([Objective-C](#user-content-objc-present-custom), [Swift](#user-content-swift-present-custom))

#### Objective-C

<a name='objc-import'></a>
##### Import LinkKit

<!-- SMARTDOWN_IMPORT_LINKKIT -->
```objc
#import <LinkKit/LinkKit.h>
```

<a name='objc-setup-shared'></a>
##### Setup Plaid Link

_Why you should setup Plaid Link iOS before presenting the `PLKPlaidLinkViewController`._

> Because when Link iOS is run it checks if the configured `public_key` is eligible for API use.
These network requests can take a while due to latency or other network related issues.
Triggering these network requests on application launch using the setup methods provided by `PLKPlaidLink`
minimizes the perceived waiting time for the user,

We recommend to setup Plaid Link early in the application's life cycle, possibly `-application:didFinishLaunchingWithOptions:`

<!-- SMARTDOWN_SETUP_SHARED -->
```objc
// With shared configuration from Info.plist
[PLKPlaidLink setupWithSharedConfiguration:^(BOOL success, NSError * _Nullable error) {
    if (success) {
        // Handle success here, e.g. by posting a notification
        NSLog(@"Plaid Link setup was successful");
        [[NSNotificationCenter defaultCenter] postNotificationName:@"PLDPlaidLinkSetupFinished" object:self];
    }
    else {
        NSLog(@"Unable to setup Plaid Link due to: %@", [error localizedDescription]);
    }
}];
```

<a name='objc-setup-custom'></a>

<!-- SMARTDOWN_SETUP_CUSTOM -->
```objc
// With custom configuration
PLKConfiguration* linkConfiguration;
@try {
    linkConfiguration = [[PLKConfiguration alloc] initWithKey:@"<#YOUR_PLAID_PUBLIC_KEY#>"
                                                          env:PLKEnvironmentDevelopment
                                                      product:PLKProductAuth];
    linkConfiguration.clientName = @"Link Demo";
    [PLKPlaidLink setupWithConfiguration:linkConfiguration completion:^(BOOL success, NSError * _Nullable error) {
        if (success) {
            // Handle success here, e.g. by posting a notification
            NSLog(@"Plaid Link setup was successful");
            [[NSNotificationCenter defaultCenter] postNotificationName:@"PLDPlaidLinkSetupFinished" object:self];
        }
        else {
            NSLog(@"Unable to setup Plaid Link due to: %@", [error localizedDescription]);
        }
    }];
} @catch (NSException *exception) {
    NSLog(@"Invalid configuration: %@", exception);
}
```

<a name='objc-protocol'></a>
##### Adopt the PLKPlaidLinkViewDelegate Protocol

<!-- SMARTDOWN_PROTOCOL -->
```objc
@interface ViewController (PLKPlaidLinkViewDelegate) <PLKPlaidLinkViewDelegate>
@end
```

<a name='objc-delegate'></a>
##### Implement the delegate methods

The `-linkViewController:didSucceedWithPublicToken:metadata:` delegate method is called
when the user successfully linked their bank account with Plaid.

<!-- SMARTDOWN_DELEGATE_SUCCESS -->
```objc
- (void)linkViewController:(PLKPlaidLinkViewController*)linkViewController
 didSucceedWithPublicToken:(NSString*)publicToken
                  metadata:(NSDictionary<NSString*,id>* _Nullable)metadata {
    [self dismissViewControllerAnimated:YES completion:^{
        // Handle success, e.g. by storing publicToken with your service
        NSLog(@"Successfully linked account!\npublicToken: %@\nmetadata: %@", publicToken, metadata);
        [self handleSuccessWithToken:publicToken metadata:metadata];
    }];
}
```

The `-linkViewController:didExitWithError:metadata` when the user initiated an exit or when an error occurred.

<!-- SMARTDOWN_DELEGATE_EXIT -->
```objc
- (void)linkViewController:(PLKPlaidLinkViewController*)linkViewController
          didExitWithError:(NSError* _Nullable)error
                  metadata:(NSDictionary<NSString*,id>* _Nullable)metadata {
    [self dismissViewControllerAnimated:YES completion:^{
        if (error) {
            NSLog(@"Failed to link account due to: %@\nmetadata: %@", [error localizedDescription], metadata);
            [self handleError:error metadata:metadata];
        }
        else {
            NSLog(@"Plaid link exited with metadata: %@", metadata);
            [self handleExitWithMetadata:metadata];
        }
    }];
}
```

 The optional `-linkViewController:didHandleEvent:metadata` delegate method is called when certain events in the Link flow have occurred. This enables your application to gain further insight into what is going on as the user goes through the Link flow.
For details about the events see the link-web onEvent [documentation](https://plaid.com/docs/api/#onevent-callback).

<!-- SMARTDOWN_DELEGATE_EVENT -->
```objc
- (void)linkViewController:(PLKPlaidLinkViewController*)linkViewController
            didHandleEvent:(NSString*)event
                  metadata:(NSDictionary<NSString*,id>* _Nullable)metadata {
    NSLog(@"Link event: %@\nmetadata: %@", event, metadata);
}
```

<a name='metadata-details'></a>
The `metadata` contains the following keys, note that values can be `[NSNull null]`.

| Constant                           | Type       | Description                                                                                                                                                              |
| ---                                | ---        | ---                                                                                                                                                                      |
| `kPLKMetadataAccountIdKey`         | String     | Identifier for the selected account                                                                                                                                      |
| `kPLKMetadataAccountKey`           | Dictionary | Contains the keys `kPLKMetadataIdKey` and `kPLKMetadataNameKey`. Only applicable when the [`selectAccount`](#user-content-config-select-account) property is set to true |
| `kPLKMetadataIdKey`                | String     | Identifier for the selected account                                                                                                                                      |
| `kPLKMetadataNameKey`              | String     | Name of the institution or selected account                                                                                                                              |
| `kPLKMetadataInstitutionKey`       | Dictionary | Contains the keys `kPLKMetadataNameKey` and `kPLKMetadataInstitutionTypeKey`                                                                                             |
| `kPLKMetadataInstitutionTypeKey`   | String     | An internal identifier for the institution                                                                                                                               |
| `kPLKMetadataStatusKey`            | String     | Indicates the point at which the user exited the Link flow (see [explanation of possible values](#user-content-metadata-status)).                                        |
| `kPLKMetadataRequestIdKey`         | String     | An internal identifier that helps us track your request in our system, please include it in every support request                                                        |
| `kPLKMetadataPlaidApiRequestIdKey` | String     | An internal identifier that helps us track your request in our system, please include it in every support request                                                        |

In addition to the above the `metadata` contains the following keys for `linkViewController:didHandleEvent:metadata`, note that values can be `[NSNull null]`.

| Constant                                | Type   | Description                                                                                                                                   |
| ---                                     | ---    | ---                                                                                                                                           |
| `kPLKMetadataInstitutionSearchQueryKey` | String | The query used to search for institutions.                                                                                                    |
| `kPLKMetadataInstitutionIdKey`          | String | The ID of the selected [institution][institutions].                                                                                           |
| `kPLKMetadataInstitutionNameKey`        | String | The name of the selected [institution][institutions].                                                                                         |
| `kPLKMetadataInstitution_TypeKey`       | String | The type of the institution (only set when using the legacy API).                                                                             |
| `kPLKMetadataLinkRequestIdKey`          | String | The request ID for the last request made by Link. This can be shared with Plaid Support to expedite investigation.                            |
| `kPLKMetadataLinkSessionIdKey`          | String | The `link_session_id` is a unique identifier for a single session of Link. It is always available and will stay constant throughout the flow. |
| `kPLKMetadataErrorTypeKey`              | String | The [error type](#errors) that the user encountered.                                                                                          |
| `kPLKMetadataErrorCodeKey`              | String | The [error code](#errors) that the user encountered.                                                                                          |
| `kPLKMetadataErrorMessageKey`           | String | The [error message](#errors) that the user encountered.                                                                                       |
| `kPLKMetadataMFATypeKey`                | String | If set, the user has encountered one of the following MFA types: `code`, `device`, `questions`, `selections`.                                 |
| `kPLKMetadataViewNameKey`               | String | The name of the [view](#metadata-view_name-) that is being transitioned to.                                                                   |
| `kPLKMetadataTimestampKey`              | String | An [ISO 8601][iso8601] representation of when the event occurred.                                                                             |
| `kPLKMetadataExitStatusKey`             | String | The [status](#metadata-status-) key indicates the point at which the user exited the Link flow.                                               |



<a name='metadata-status'></a>
Possible values for the `kPLKMetadataStatusKey` metadata:

| Constant                        | Description                                                                                                                 |
| ---                             | ---                                                                                                                         |
| `kPLKStatusConnected`           | User completed the Link flow                                                                                                |
| `kPLKStatusRequiresQuestions`   | User prompted to answer security question(s)                                                                                |
| `kPLKStatusRequiresSelections`  | User prompted to answer multiple choice question(s)                                                                         |
| `kPLKStatusRequiresCode`        | User prompted to provide a one-time passcode.                                                                               |
| `kPLKStatusChooseDevice`        | User prompted to select a device at which to receive a one-time passcode.                                                   |
| `kPLKStatusRequiresCredentials` | User prompted to provide credentials for the selected financial institution or has not yet selected a financial institution |
| `kPLKStatusRequiresRecaptcha`   | User prompted to verify they are human via reCAPTCHA                                                                        |
| `kPLKStatusInstitutionNotFound` | User exited the Link flow after unsuccessfully (no results returned) searching for a financial institution                  |


<a name='objc-present-shared'></a>
##### Present `PLKPlaidLinkViewController`

Starting the Plaid Link iOS experience is as simple as presenting an instance
of `PLKPlaidLinkViewController`.
<!-- SMARTDOWN_PRESENT_SHARED -->
```objc
// With shared configuration from Info.plist
id<PLKPlaidLinkViewDelegate> linkViewDelegate  = self;
PLKPlaidLinkViewController* linkViewController = [[PLKPlaidLinkViewController alloc] initWithDelegate:linkViewDelegate];
if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
    linkViewController.modalPresentationStyle = UIModalPresentationFormSheet;
}
[self presentViewController:linkViewController animated:YES completion:nil];
```

<a name='objc-present-custom'></a>

When you would like to have more control over the configuration during runtime you can
create an instance of `PLKConfiguration` and pass that during the
initialisation of the `PLKPlaidLinkViewController`.

<!-- SMARTDOWN_PRESENT_CUSTOM -->
```objc
// With custom configuration
PLKConfiguration* linkConfiguration;
@try {
    linkConfiguration = [[PLKConfiguration alloc] initWithKey:@"<#YOUR_PLAID_PUBLIC_KEY#>" env:PLKEnvironmentSandbox product:PLKProductAuth];
    linkConfiguration.clientName = @"Link Demo";
    id<PLKPlaidLinkViewDelegate> linkViewDelegate  = self;
    PLKPlaidLinkViewController* linkViewController = [[PLKPlaidLinkViewController alloc] initWithConfiguration:linkConfiguration delegate:linkViewDelegate];
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
        linkViewController.modalPresentationStyle = UIModalPresentationFormSheet;
    }
    [self presentViewController:linkViewController animated:YES completion:nil];
} @catch (NSException *exception) {
    NSLog(@"Invalid configuration: %@", exception);
}
```

<a name='custom-initializer-info'></a>
##### Custom Initializers

To preselect an institution instantiate the `PLKPlaidLinkViewController` object using
[`initWithInstitution:delegate:`](https://github.com/plaid/link/blob/master/ios/LinkKit.framework/Headers/PLKPlaidLinkViewController.h#L80-L96)
and pass the `institution_id` (e.g. `ins_109509`) of the institution you would
like to use for as the first parameter. Then present the `linkViewController` instance as usual.

Refer to the [Plaid API documentation](https://plaid.com/docs/api/#institution-search)
on how to find out the `institution_id` for an institution.

<a name='objc-custom-initializer'></a>
<!-- SMARTDOWN_CUSTOM_INITIALIZER -->
```objc
id<PLKPlaidLinkViewDelegate> linkViewDelegate  = self;
PLKPlaidLinkViewController* linkViewController = [[PLKPlaidLinkViewController alloc] initWithInstitution:@"<#INSTITUTION_ID#>" delegate:linkViewDelegate];
if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
    linkViewController.modalPresentationStyle = UIModalPresentationFormSheet;
}
[self presentViewController:linkViewController animated:YES completion:nil];
```

<a name='update-mode-info'></a>
##### Update Mode

To initiate the [update mode][link-update-mode] instantiate the `PLKPlaidLinkViewController`
object using [`initWithPublicToken:delegate:`](https://github.com/plaid/link/blob/master/ios/LinkKit.framework/Headers/PLKPlaidLinkViewController.h#L120-L136)
and pass your [generated `public_token`][create-public-token] as the first
parameter. Then present the `linkViewController` instance as usual.

<a name='objc-update-mode'></a>
<!-- SMARTDOWN_UPDATE_MODE -->
```objc
id<PLKPlaidLinkViewDelegate> linkViewDelegate  = self;
PLKPlaidLinkViewController* linkViewController = [[PLKPlaidLinkViewController alloc] initWithPublicToken:@"<#GENERATED_PUBLIC_TOKEN#>" delegate:linkViewDelegate];
if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) {
    linkViewController.modalPresentationStyle = UIModalPresentationFormSheet;
}
[self presentViewController:linkViewController animated:YES completion:nil];
```

#### Swift

<a name='swift-import'></a>
##### Import LinkKit

<!-- SMARTDOWN_IMPORT_LINKKIT -->
```swift
import LinkKit
```

<a name='swift-setup-shared'></a>
##### Setup Plaid Link

Be sure to read [why you should setup Plaid Link iOS before presenting the `PLKPlaidLinkViewController`](#setup-plaid-link).

<!-- SMARTDOWN_SETUP_SHARED -->
```swift
// With shared configuration from Info.plist
PLKPlaidLink.setup { (success, error) in
    if (success) {
        // Handle success here, e.g. by posting a notification
        NSLog("Plaid Link setup was successful")
        NotificationCenter.default.post(name: NSNotification.Name(rawValue: "PLDPlaidLinkSetupFinished"), object: self)
    }
    else if let error = error {
        NSLog("Unable to setup Plaid Link due to: \(error.localizedDescription)")
    }
    else {
        NSLog("Unable to setup Plaid Link")
    }
}
```

<a name='swift-setup-custom'></a>
<!-- SMARTDOWN_SETUP_CUSTOM -->
```swift
// With custom configuration
let linkConfiguration = PLKConfiguration(key: "<#YOUR_PLAID_PUBLIC_KEY#>", env: .development, product: .auth)
linkConfiguration.clientName = "Link Demo"
PLKPlaidLink.setup(with: linkConfiguration) { (success, error) in
    if (success) {
        // Handle success here, e.g. by posting a notification
        NSLog("Plaid Link setup was successful")
        NotificationCenter.default.post(name: NSNotification.Name(rawValue: "PLDPlaidLinkSetupFinished"), object: self)
    }
    else if let error = error {
        NSLog("Unable to setup Plaid Link due to: \(error.localizedDescription)")
    }
    else {
        NSLog("Unable to setup Plaid Link")
    }
}
```

<a name='swift-protocol'></a>
##### Adopt the PLKPlaidLinkViewDelegate Protocol

<!-- SMARTDOWN_PROTOCOL -->
```swift
extension ViewController : PLKPlaidLinkViewDelegate
```

<a name='swift-delegate'></a>
##### Implement the delegate methods

Be sure to read about the details regarding the [metadata](#user-content-metadata-details).

<!-- SMARTDOWN_DELEGATE_SUCCESS -->
```swift
func linkViewController(_ linkViewController: PLKPlaidLinkViewController, didSucceedWithPublicToken publicToken: String, metadata: [String : Any]?) {
    dismiss(animated: true) {
        // Handle success, e.g. by storing publicToken with your service
        NSLog("Successfully linked account!\npublicToken: \(publicToken)\nmetadata: \(metadata ?? [:])")
        self.handleSuccessWithToken(publicToken, metadata: metadata)
    }
}
```

<!-- SMARTDOWN_DELEGATE_EXIT -->
```swift
func linkViewController(_ linkViewController: PLKPlaidLinkViewController, didExitWithError error: Error?, metadata: [String : Any]?) {
    dismiss(animated: true) {
        if let error = error {
            NSLog("Failed to link account due to: \(error.localizedDescription)\nmetadata: \(metadata ?? [:])")
            self.handleError(error, metadata: metadata)
        }
        else {
            NSLog("Plaid link exited with metadata: \(metadata ?? [:])")
            self.handleExitWithMetadata(metadata)
        }
    }
}
```

<a name='swift-present-shared'></a>
##### Present `PLKPlaidLinkViewController`

<!-- SMARTDOWN_PRESENT_SHARED -->
```swift
// With shared configuration from Info.plist
let linkViewDelegate = self
let linkViewController = PLKPlaidLinkViewController(delegate: linkViewDelegate)
if (UI_USER_INTERFACE_IDIOM() == .pad) {
    linkViewController.modalPresentationStyle = .formSheet;
}
present(linkViewController, animated: true)
```

<a name='swift-present-custom'></a>
<!-- SMARTDOWN_PRESENT_CUSTOM -->
```swift
// With custom configuration
let linkConfiguration = PLKConfiguration(key: "<#YOUR_PLAID_PUBLIC_KEY#>", env: .sandbox, product: .auth)
linkConfiguration.clientName = "Link Demo"
let linkViewDelegate = self
let linkViewController = PLKPlaidLinkViewController(configuration: linkConfiguration, delegate: linkViewDelegate)
if (UI_USER_INTERFACE_IDIOM() == .pad) {
    linkViewController.modalPresentationStyle = .formSheet;
}
present(linkViewController, animated: true)
```

##### Custom Initializers

For more information please read [additional information about custom initializers](#user-content-custom-initializer-info)

<a name='swift-custom-initializer'></a>
<!-- SMARTDOWN_CUSTOM_INITIALIZER -->
```swift
let linkViewDelegate = self
let linkViewController = PLKPlaidLinkViewController(institution: "<#INSTITUTION_ID#>", delegate: linkViewDelegate)
if (UI_USER_INTERFACE_IDIOM() == .pad) {
    linkViewController.modalPresentationStyle = .formSheet;
}
present(linkViewController, animated: true)
```

##### Update Mode

For more information please read [additional information about update mode](#user-content-update-mode-info)

<a name='swift-update-mode'></a>
<!-- SMARTDOWN_UPDATE_MODE -->
```swift
let linkViewDelegate = self
let linkViewController = PLKPlaidLinkViewController(publicToken: "<#GENERATED_PUBLIC_TOKEN#>", delegate: linkViewDelegate)
if (UI_USER_INTERFACE_IDIOM() == .pad) {
    linkViewController.modalPresentationStyle = .formSheet;
}
present(linkViewController, animated: true)
```

### Customization

:warning: **The preferred and recommended method to customize LinkKit is to use customization feature in the [dashboard][dashboard-customization], LinkKit will
automatically use the values you provide there.**

Customization allows you to change the text of certain user interface elements in the Link flow ([see below](#user-content-link-customization)).
In the rare case where customization is necessary from within your application directly, either add your customizations to the `PLKPlaidLinkConfiguration` section in your
application's Info.plist ([example](#shared-configuration-customization)) or call the `customizeWithDictionary:` method passing in the desired customizations
([example](#instance-configuration-customization)) on the `PLKConfiguration` object that is used to configure LinkKit.

<a name='link-customization'>The following images illustrate the customizable elements on different panes in detail:</a>

<img src='docs/images/customization/connectedPane.png' width='250' title='Connected pane customization'>
<img src='docs/images/customization/reconnectedPane.png' width='250' title='Reconnected pane customization'>
<img src='docs/images/customization/institutionSelectPane.png' width='250' title='Institution select pane customization'>
<img src='docs/images/customization/institutionSearchPane_initialMessage.png' width='250' title='Institution Search pane customization'>
<img src='docs/images/customization/institutionSearchPane_noResultsMessage.png' width='250' title='Institution Search pane customization'>

The following table shows which elements can be customized on which panes:

| ↓UI Element \ Pane→ | `connectedPane` | `reconnectedPane` | `institutionSelectPane` | `institutionSearchPane` |
| ---                 | :---:           | :---:             | :---:                   | :---:                   |
| `title`             | ✓               | ✓                 | ✓                       | —                       |
| `message`           | ✓¹              | ✓¹                | —                       | —                       |
| `submitButton`      | ✓               | ✓                 | —                       | —                       |
| `searchButton`      | —               | —                 | ✓                       | —                       |
| `initialMessage`    | —               | —                 | —                       | ✓                       |
| `noResultsMessage`  | —               | —                 | —                       | ✓                       |
| `exitButton`        | —               | —                 | —                       | ✓                       |

¹ Any occurrences of `<CLIENT>` in the `message` are replaced with the configured `clientName`.

✓ Customization supported, — Customization not supported

#### Shared Configuration Customization

To customize LinkKit using a shared configuration from your application's `Info.plist` add a `customization` dictionary
to the `PLKPlaidLinkConfiguration` section and add the Link Panes with the UI Elements you would like to customize:

| Key                     | Type       | Allowed Values                                     |
| ---                     | ---        | ---                                                |
| **Link Panes**          |
| `connectedPane`         | Dictionary | `title`, `message`, `submitButton`                 |
| `reconnectedPane`       | Dictionary | `title`, `message`, `submitButton`                 |
| `institutionSelectPane` | Dictionary | `title`, `searchButton`                            |
| `institutionSearchPane` | Dictionary | `initialMessage`, `noResultsMessage`, `exitButton` |
| **UI Elements**         |
| `title`                 | String     |                                                    |
| `messsage`              | String     |                                                    |
| `submitButton`          | String     |                                                    |
| `searchButton`          | String     |                                                    |
| `initialMessage`        | String     |                                                    |
| `noResultsMessage`      | String     |                                                    |
| `exitButton`            | String     |                                                    |

An example making use of all available customization with a LinkKit shared configuration looks as follows:

<img src='docs/images/customization/setupPlistCustomization.png' width='350' title='Adding copy customization to Info.plist'>

As a template you can copy and paste the following XML below the `PLKPlaidLinkConfiguration` key in the `Info.plist`
and rename the `New item` to `customization`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>connectedPane</key>
	<dict>
		<key>title</key>
		<string>Sign-up successful</string>
		<key>message</key>
		<string>You successfully linked your account with &lt;CLIENT&gt;</string>
		<key>submitButton</key>
		<string>Continue</string>
	</dict>
	<key>reconnectedPane</key>
	<dict>
		<key>title</key>
		<string>Update successful</string>
		<key>message</key>
		<string>You successfully updated your account credentials with &lt;CLIENT&gt;</string>
		<key>submitButton</key>
		<string>Continue</string>
	</dict>
	<key>institutionSelectPane</key>
	<dict>
		<key>title</key>
		<string>Choose your bank</string>
		<key>searchButton</key>
		<string>Search for your bank</string>
	</dict>
	<key>institutionSearchPane</key>
	<dict>
		<key>exitButton</key>
		<string>Quit</string>
		<key>initialMessage</key>
		<string>Find your bank or credit union</string>
		<key>noResultsMessage</key>
		<string>Unfortunately the institution you searched for could not be found</string>
	</dict>
</dict>
</plist>
```

#### Instance Configuration Customization

As mentioned above you can add customizations to your instance configuration by calling `customizeWithDictionary:`
on your `PLKConfiguration` object and provide just the customizations you need.
An example making use of all available customization with a LinkKit instance configuration looks as follows

<!-- SMARTDOWN_CUSTOMIZATION -->

for Objective-C:
```objc
[linkConfiguration customizeWithDictionary:@{
                                 kPLKConnectedPaneKey: @{
                                         kPLKCustomizationTitleKey: @"Sign-up successful",
                                         kPLKCustomizationMessageKey: @"You successfully linked your account with <CLIENT>",
                                         kPLKCustomizationSubmitButtonKey: @"Continue"
                                         },
                                 
                                 kPLKReconnectedPaneKey: @{
                                         kPLKCustomizationTitleKey: @"Update successful",
                                         kPLKCustomizationMessageKey: @"You successfully updated your account credentials <CLIENT>",
                                         kPLKCustomizationSubmitButtonKey: @"Continue"
                                         },
                                 
                                 kPLKInstitutionSelectPaneKey: @{
                                         kPLKCustomizationTitleKey: @"Choose your bank",
                                         kPLKCustomizationSearchButtonKey: @"Search for your bank"
                                         },
                                 
                                 kPLKInstitutionSearchPaneKey: @{
                                         kPLKCustomizationExitButtonKey: @"Quit",
                                         kPLKCustomizationInitialMessageKey: @"Find your bank or credit union",
                                         kPLKCustomizationNoResultsMessageKey: @"Unfortunately the institution you searched for could not be found"
                                         },
                                 }];
```

and Swift:
```swift
linkConfiguration.customize(with: [
    kPLKConnectedPaneKey: [
        kPLKCustomizationTitleKey: "Sign-up successful",
        kPLKCustomizationMessageKey: "You successfully linked your account with <CLIENT>",
        kPLKCustomizationSubmitButtonKey: "Continue"
    ],
    
    kPLKReconnectedPaneKey: [
        kPLKCustomizationTitleKey: "Update successful",
        kPLKCustomizationMessageKey: "You successfully updated your account credentials <CLIENT>",
        kPLKCustomizationSubmitButtonKey: "Continue"
    ],
    
    kPLKInstitutionSelectPaneKey: [
        kPLKCustomizationTitleKey: "Choose your bank",
        kPLKCustomizationSearchButtonKey: "Search for your bank"
    ],
    
    kPLKInstitutionSearchPaneKey: [
        kPLKCustomizationExitButtonKey: "Quit",
        kPLKCustomizationInitialMessageKey: "Find your bank or credit union",
        kPLKCustomizationNoResultsMessageKey: "Unfortunately the institution you searched for could not be found"
    ],
    ])
```

NSString constants for the customization pane and element names are available:

| Customizable Pane Name  | Constant                       |
| ---                     | ---                            |
| `connectedPane`         | `kPLKConnectedPaneKey`         |
| `reconnectedPane`       | `kPLKReconnectedPaneKey`       |
| `institutionSelectPane` | `kPLKInstitutionSelectPaneKey` |
| `institutionSearchPane` | `kPLKInstitutionSearchPaneKey` |

| Customizable Element Name | Constant                               |
| ---                       | ---                                    |
| `title`                   | `kPLKCustomizationTitleKey`            |
| `message`                 | `kPLKCustomizationMessageKey`          |
| `submitButton`            | `kPLKCustomizationSubmitButtonKey`     |
| `searchButton`            | `kPLKCustomizationSearchButtonKey`     |
| `initialMessage`          | `kPLKCustomizationInitialMessageKey`   |
| `noResultsMessage`        | `kPLKCustomizationNoResultsMessageKey` |
| `exitButton`              | `kPLKCustomizationExitButtonKey`       |


### Troubleshooting

When things work differently as expected LinkKit will use the value of the
`PLKPLAIDLINK_DIAGNOSTICS` environment variable and enable some logging to
the console. **Environment Variables** can be added in the **Arguments** tab
of the **Run** phase in the Scheme Editor (Product ► Scheme ► Edit Scheme `⌘<`)

<img src='docs/images/setup/09-edit_scheme.jpg' width='350' title='Edit current scheme'>
<img src='docs/images/setup/09-troubleshooting.jpg' width='350' title='Setting the PLKPLAIDLINK_DIAGNOSTICS environment variable'>

Available settings are:

| Loglevel | `PLKPLAIDLINK_DIAGNOSTICS` Value |
| ---      | :---:                            |
| None     | 0                                |
| Error    | 1                                |
| Info     | 2                                |
| Debug    | 3                                |

### About the LinkDemo Xcode projects

In order to compile the source code that uses the [custom configuration](#configuration)
add `-DUSE_CUSTOM_CONFIG` to `OTHER_SWIFT_FLAGS` in the
LinkDemo-Swift build settings and to `OTHER_CFLAGS` in the LinkDemo-ObjC build settings.

<img src='docs/images/setup/10-use_custom_config.jpg' width='350' title='Enabling use of custom configuration'>

Throughout the source code there are HTML-like comments such as
<code>&lt;!-- SMARTDOWN_PRESENT_CUSTOM --&gt;</code>, they are used to update
the code examples in this README from the actual source code ensuring that the
examples are working as intended.


## Known issues


[linkkit]: LinkKit.framework
[dashboard-keys]: https://dashboard.plaid.com/account/keys
[dashboard-customization]: https://dashboard.plaid.com/link
[link-update-mode]: https://plaid.com/docs/api/#updating-items-via-link
[create-public-token]: https://plaid.com/docs/api/#creating-public-tokens
