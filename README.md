# Watson Developer Cloud Python SDK

[![Build Status](https://travis-ci.org/watson-developer-cloud/python-sdk.svg?branch=master)](https://travis-ci.org/watson-developer-cloud/python-sdk)
[![Slack](https://wdc-slack-inviter.mybluemix.net/badge.svg)](https://wdc-slack-inviter.mybluemix.net)
[![Latest Stable Version](https://img.shields.io/pypi/v/ibm-watson.svg)](https://pypi.python.org/pypi/ibm-watson)
[![CLA assistant](https://cla-assistant.io/readme/badge/watson-developer-cloud/python-sdk)](https://cla-assistant.io/watson-developer-cloud/python-sdk)

Python client library to quickly get started with the various [Watson APIs][wdc] services.

<details>
  <summary>Table of Contents</summary>

  * [Before you begin](#before-you-begin)
  * [Installation](#installation)
  * [Examples](#examples)
  * [Running in IBM Cloud](#running-in-ibm-cloud)
  * [Authentication](#authentication)
    * [Getting credentials](#getting-credentials)
    * [IAM](#iam)
    * [Username and password](#username-and-password)
  * [Python version](#python-version)
  * [Changes for v1.0](#changes-for-v10)
  * [Changes for v2.0](#changes-for-v20)
  * [Changes for v3.0](#changes-for-v30)
  * [Migration](#migration)
  * [Configuring the http client](#configuring-the-http-client-supported-from-v110)
  * [Disable SSL certificate verification](#disable-ssl-certificate-verification)
  * [Sending request headers](#sending-request-headers)
  * [Parsing HTTP response info](#parsing-http-response-info)
  * [Using Websockets](#using-websockets)
  * [IBM Cloud Pak for Data(ICP4D)](#ibm-cloud-pak-for-data(icp4d))
  * [Dependencies](#dependencies)
  * [License](#license)
  * [Contributing](#contributing)
  * [Featured Projects](#featured-projects)

</details>

## Before you begin
* You need an [IBM Cloud][ibm-cloud-onboarding] account.

## Installation
To install, use `pip` or `easy_install`:

```bash
pip install --upgrade ibm-watson
```

or

```bash
easy_install --upgrade ibm-watson
```

Note the following:
a) Versions prior to 3.0.0 can be installed using:

```bash
pip install --upgrade watson-developer-cloud
```

b) If you run into permission issues try:

```bash
sudo -H pip install --ignore-installed six ibm-watson
```

For more details see [#225](https://github.com/watson-developer-cloud/python-sdk/issues/225)

c) In case you run into problems installing the SDK in DSX, try
```
!pip install --upgrade pip
```
Restarting the kernel

For more details see [#405](https://github.com/watson-developer-cloud/python-sdk/issues/405)

## Examples

The [examples][examples] folder has basic and advanced examples. The examples within each service assume that you already have [service credentials](#getting-credentials).

## Running in IBM Cloud

If you run your app in IBM Cloud, the SDK gets credentials from the [`VCAP_SERVICES`][vcap_services] environment variable.

## Authentication

Watson services are migrating to token-based Identity and Access Management (IAM) authentication.

- With some service instances, you authenticate to the API by using **[IAM](#iam)**.
- In other instances, you authenticate by providing the **[username and password](#username-and-password)** for the service instance.

**Note:** Authenticating with the X-Watson-Authorization-Token header is deprecated. The token continues to work with Cloud Foundry services, but is not supported for services that use Identity and Access Management (IAM) authentication. See [here](#iam) for details.

### Getting credentials
To find out which authentication to use, view the service credentials. You find the service credentials for authentication the same way for all Watson services:

1. Go to the IBM Cloud [Dashboard](https://cloud.ibm.com/) page.
1. Either click an existing Watson service instance in your [resource list](https://cloud.ibm.com/resources) or click [**Create resource > AI**](https://cloud.ibm.com/catalog?category=ai) and create a service instance.
1. Click on the **Manage** item in the left nav bar of your service instance.

On this page, you should be able to see your credentials for accessing your service instance.

### Supplying credentials

There are two ways to supply the credentials you found above to the SDK for authentication.

#### Credential file (easier!)

With a credential file, you just need to put the file in the right place and the SDK will do the work of parsing and authenticating. You can get this file by clicking the **Download** button for the credentials in the **Manage** tab of your service instance.

The file downloaded will be called `ibm-credentials.env`. This is the name the SDK will search for and **must** be preserved unless you want to configure the file path (more on that later). The SDK will look for your `ibm-credentials.env` file in the following places (in order):

- Your system's home directory
- The top-level directory of the project you're using the SDK in

As long as you set that up correctly, you don't have to worry about setting any authentication options in your code. So, for example, if you created and downloaded the credential file for your Discovery instance, you just need to do the following:

```python
discovery = DiscoveryV1(version='2018-08-01')
```

And that's it!

If you're using more than one service at a time in your code and get two different `ibm-credentials.env` files, just put the contents together in one `ibm-credentials.env` file and the SDK will handle assigning credentials to their appropriate services.

If you would like to configure the location/name of your credential file, you can set an environment variable called `IBM_CREDENTIALS_FILE`. **This will take precedence over the locations specified above.** Here's how you can do that:

```bash
export IBM_CREDENTIALS_FILE="<path>"
```

where `<path>` is something like `/home/user/Downloads/<file_name>.env`.

#### Manually
If you'd prefer to set authentication values manually in your code, the SDK supports that as well. The way you'll do this depends on what type of credentials your service instance gives you.

### IAM

IBM Cloud has migrated to token-based Identity and Access Management (IAM) authentication. IAM authentication uses a service API key to get an access token that is passed with the call. Access tokens are valid for approximately one hour and must be regenerated.

You supply either an IAM service **API key** or an **access token**:

- Use the API key to have the SDK manage the lifecycle of the access token. The SDK requests an access token, ensures that the access token is valid, and refreshes it if necessary.
- Use the access token if you want to manage the lifecycle yourself. For details, see [Authenticating with IAM tokens](https://cloud.ibm.com/docs/services/watson?topic=watson-iam).
- Use a server-side to generate access tokens using your IAM API key for untrusted environments like client-side scripts. The generated access tokens will be valid for one hour and can be refreshed.

### Generating access tokens using IAM API key
```python
# In your API endpoint use this to generate new access tokens
iam_token_manager = IAMTokenManager(iam_apikey='<apikey>')
token = iam_token_manager.get_token()
```

#### Supplying the IAM API key

```python
# In the constructor, letting the SDK manage the IAM token
discovery = DiscoveryV1(version='2018-08-01',
                        url='<url_as_per_region>',
                        apikey='<apikey>',
                        iam_url='<iam_url>') # optional - the default value is https://iam.cloud.ibm.com/identity/token
```

```python
# after instantiation, letting the SDK manage the IAM token
discovery = DiscoveryV1(version='2018-08-01', url='<url_as_per_region>')
discovery.set_apikey('<apikey>')
```

#### Supplying the access token
```python
# in the constructor, assuming control of managing IAM token
discovery = DiscoveryV1(version='2018-08-01',
                        url='<url_as_per_region>',
                        iam_access_token='<iam_access_token>')
```

```python
# after instantiation, assuming control of managing IAM token
discovery = DiscoveryV1(version='2018-08-01', url='<url_as_per_region>')
discovery.set_iam_access_token('<access_token>')
```

### Username and password
```python
from ibm_watson import DiscoveryV1
# In the constructor
discovery = DiscoveryV1(version='2018-08-01', url='<url_as_per_region>', username='<username>', password='<password>')
```

```python
# After instantiation
discovery = DiscoveryV1(version='2018-08-01', url='<url_as_per_region>')
discovery.set_username_and_password('<username>', '<password>')
```

## Python version

Tested on Python 2.7, 3.5, 3.6, and 3.7.

## Changes for v1.0
Version 1.0 focuses on the move to programmatically-generated code for many of the services. See the [changelog](https://github.com/watson-developer-cloud/python-sdk/wiki/Changelog) for the details.

## Changes for v2.0
`DetailedResponse` which contains the result, headers and HTTP status code is now the default response for all methods.
```python
from ibm_watson import AssistantV1

assistant = AssistantV1(
    username='xxx',
    password='yyy',
    url='<url_as_per_region>',
    version='2018-07-10')

response = assistant.list_workspaces(headers={'Custom-Header': 'custom_value'})
print(response.get_result())
print(response.get_headers())
print(response.get_status_code())
```
See the [changelog](https://github.com/watson-developer-cloud/python-sdk/wiki/Changelog) for the details.

## Changes for v3.0
The SDK is generated using OpenAPI Specification(OAS3). Changes are basic reordering of parameters in function calls.

The package is renamed to ibm_watson. See the [changelog](https://github.com/watson-developer-cloud/python-sdk/wiki/Changelog) for the details.

## Migration
This version includes many breaking changes as a result of standardizing behavior across the new generated services. Full details on migration from previous versions can be found [here](https://github.com/watson-developer-cloud/python-sdk/wiki/Migration).

## Configuring the http client (Supported from v1.1.0)
To set client configs like timeout use the `with_http_config()` function and pass it a dictionary of configs.

```python
from ibm_watson import AssistantV1

assistant = AssistantV1(
    username='xxx',
    password='yyy',
    url='<url_as_per_region>',
    version='2018-07-10')

assistant.set_http_config({'timeout': 100})
response = assistant.message(workspace_id=workspace_id, input={
    'text': 'What\'s the weather like?'}).get_result()
print(json.dumps(response, indent=2))
```

## Disable SSL certificate verification
For ICP(IBM Cloud Private), you can disable the SSL certificate verification by:

```python
service.disable_SSL_verification()
```

## Sending request headers
Custom headers can be passed in any request in the form of a `dict` as:
```python
headers = {
    'Custom-Header': 'custom_value'
}
```
For example, to send a header called `Custom-Header` to a call in Watson Assistant, pass
the headers parameter as:
```python
from ibm_watson import AssistantV1

assistant = AssistantV1(
    username='xxx',
    password='yyy',
    url='<url_as_per_region>',
    version='2018-07-10')

response = assistant.list_workspaces(headers={'Custom-Header': 'custom_value'}).get_result()
```

## Parsing HTTP response info
If you would like access to some HTTP response information along with the response model, you can set the `set_detailed_response()` to `True`. Since Python SDK `v2.0`, it is set to `True`
```python
from ibm_watson import AssistantV1

assistant = AssistantV1(
    username='xxx',
    password='yyy',
    url='<url_as_per_region>',
    version='2018-07-10')

assistant.set_detailed_response(True)
response = assistant.list_workspaces(headers={'Custom-Header': 'custom_value'}).get_result()
print(response)
```

This would give an output of `DetailedResponse` having the structure:
```python
{
    'result': <response returned by service>,
    'headers': { <http response headers> },
    'status_code': <http status code>
}
```
You can use the `get_result()`, `get_headers()` and get_status_code() to return the result, headers and status code respectively.

## Using Websockets
The Text to Speech service supports synthesizing text to spoken audio using web sockets with the `synthesize_using_websocket`. The Speech to Text service supports recognizing speech to text using web sockets with the `recognize_using_websocket`. These methods need a custom callback class to listen to events. Below is an example of `synthesize_using_websocket`. Note: The service accepts one request per connection.

```py
from ibm_watson.websocket import SynthesizeCallback

class MySynthesizeCallback(SynthesizeCallback):
    def __init__(self):
        SynthesizeCallback.__init__(self)

    def on_audio_stream(self, audio_stream):
        return audio_stream

    def on_data(self, data):
        return data

my_callback = MySynthesizeCallback()
service.synthesize_using_websocket('I like to pet dogs',
                                   my_callback,
                                   accept='audio/wav',
                                   voice='en-US_AllisonVoice'
                                  )
```

## IBM Cloud Pak for Data(ICP4D)
If your service instance is of ICP4D, below are two ways of initializing the assistant service.

#### 1) Supplying the `username`, `password`, `icp4d_url` and `authentication_type`
The SDK will manage the token for the user
```python
assistant = AssistantV1(
    version='<version',
    username='<your username>',
    password='<your password>',
    url='<service url>', # should be of the form https://{icp_cluster_host}/{deployment}/assistant/{instance-id}/api
    icp4d_url='<authentication url>', # should be of the form https://{icp_cluster_host}
    authentication_type='icp4d')

assistant.disable_SSL_verification() # MAKE SURE SSL VERIFICATION IS DISABLED
```

#### 2) Supplying the access token
```python
assistant = AssistantV1(
    version='<version>',
    url='service url', # should be of the form https://{icp_cluster_host}/{deployment}/assistant/{instance-id}/api
    icp4d_access_token='<your managed access token>')

assistant.disable_SSL_verification() # MAKE SURE SSL VERIFICATION IS DISABLED
```

## Dependencies

* [requests]
* `python_dateutil` >= 2.5.3
* [responses] for testing
* Following for web sockets support in speech to text
   * `websocket-client` 0.48.0
* `ibm_cloud_sdk_core` >=0.5.1

## Contributing

See [CONTRIBUTING.md][CONTRIBUTING].

## Featured Projects

Here are some projects that have been using the SDK:

* [NLC ICD-10 Classifier](https://github.com/IBM/nlc-icd10-classifier)
* [Cognitive Moderator Service](https://github.com/IBM/cognitive-moderator-service)

We'd love to highlight cool open-source projects that use this SDK! If you'd like to get your project added to the list, feel free to make an issue linking us to it.


## License

This library is licensed under the [Apache 2.0 license][license].

[wdc]: http://www.ibm.com/watson/developercloud/
[ibm_cloud]: https://cloud.ibm.com/
[watson-dashboard]: https://cloud.ibm.com/catalog?category=ai
[responses]: https://github.com/getsentry/responses
[requests]: http://docs.python-requests.org/en/latest/
[examples]: https://github.com/watson-developer-cloud/python-sdk/tree/master/examples
[CONTRIBUTING]: https://github.com/watson-developer-cloud/python-sdk/blob/master/CONTRIBUTING.md
[license]: http://www.apache.org/licenses/LICENSE-2.0
[vcap_services]: https://cloud.ibm.com/docs/services/watson?topic=watson-vcapServices
[ibm-cloud-onboarding]: https://cloud.ibm.com/registration?target=/developer/watson&cm_sp=WatsonPlatform-WatsonServices-_-OnPageNavLink-IBMWatson_SDKs-_-Python
