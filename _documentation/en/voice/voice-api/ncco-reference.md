---
title: NCCO Reference
description: The Nexmo Call Control Objects used to manage your Voice API calls.
---

# NCCO reference

A Call Control Object (NCCO) is represented by a JSON array. You can use it to control the flow of a Voice API call. For your NCCO to execute correctly, the JSON objects must be valid.

While developing and testing NCCOs, you can use the Voice Playground to try out NCCOs interactively. You can [read more about it in the Voice API Overview](/voice/voice-api/overview#voice-playground) or [go directly to the Voice Playground in the Dashboard](https://dashboard.nexmo.com/voice/playground).

## NCCO actions

The order of actions in the NCCO controls the flow of the Call. Actions that have to complete before the next action can be executed are *synchronous*. Other actions are *asynchronous*. That is, they are supposed to continue over the following actions until a condition is met. For example, a `record` action terminates when the `endOnSilence` option is met. When all the actions in the NCCO are complete, the Call ends.

The NCCO actions and the options and types for each action are:

Action | Description | Synchronous
-- | -- | --
[record](#record) | All or part of a Call | No
[conversation](#conversation) | Create or join an existing [Conversation](/conversation/concepts/conversation) | Yes
[connect](#connect) | To a connectable endpoint such as a phone number or VBC extension. | Yes
[talk](#talk) | Send synthesized speech to a Conversation. | Yes, unless *bargeIn=true*
[stream](#stream) | Send audio files to a Conversation. | Yes, unless *bargeIn=true*
[input](#input) | Collect digits or capture speech input from the person you are calling. | Yes
[notify](#notify) | Send a request to your application to track progress through an NCCO | Yes
[pay](#pay) [Developer Preview] | Ask the user for card details and process payment. See the [Guide](/voice/voice-api/guides/payments) for more details | Yes

> **Note**: [Connect an inbound call](/voice/voice-api/code-snippets/connect-an-inbound-call) provides an example of how to serve your NCCOs to Vonage after a Call or Conference is initiated.

## Record

Use the `record` action to record a Call or part of a Call:

```json
[
  {
    "action": "record",
    "eventUrl": ["https://example.com/recordings"]
  },
  {
    "action": "connect",
    "eventUrl": ["https://example.com/events"],
    "from":"447700900000",
    "endpoint": [
      {
        "type": "phone",
        "number": "447700900001"
      }
    ]
  }
]
```

The record action is asynchronous. Recording starts when the record action is executed in the NCCO and finishes when the synchronous condition in the action is met. That is, `endOnSilence`, `timeOut` or `endOnKey`. If you do not set a synchronous condition, the Voice API immediately executes the next NCCO without recording.

For information about the workflow to follow, see [Recording](/voice/voice-api/guides/recording).

You can use the following options to control a `record` action:

Option | Description | Required
 -- | -- | --
`format` | Record the Call in a specific format.  Options are: <ul><li>`mp3`</li><li>`wav`</li><li>`ogg`</li></ul> The default value is `mp3`, or `wav` when recording more than 2 channels. | No
`split` | Record the sent and received audio in separate channels of a stereo recording—set to `conversation` to enable this.|No
`channels` | The number of channels to record (maximum `32`). If the number of participants exceeds `channels` any additional participants will be added to the last channel in file. split `conversation` must also be enabled.| No
`endOnSilence` | Stop recording after n seconds of silence. Once the recording is stopped the recording data is sent to `event_url`. The range of possible values is 3<=`endOnSilence`<=10. | No
`endOnKey` | Stop recording when a digit is pressed on the handset. Possible values are: `*`, `#` or any single digit e.g. `9`. | No
`timeOut` | The maximum length of a recording in seconds. One the recording is stopped the recording data is sent to `event_url`. The range of possible values is between `3` seconds and `7200` seconds (2 hours). | No
`beepStart` | Set to `true` to play a beep when a recording starts. | No
`eventUrl` | The URL to the webhook endpoint that is called asynchronously when a recording is finished. If the message recording is hosted by Vonage, this webhook contains the [URL you need to download the recording and other meta data](#recording_return_parameters). | No
`eventMethod` | The HTTP method used to make the request to `eventUrl`. The default value is `POST`. | No

<a name="recording_return_parameters"></a>
The following example shows the return parameters sent to `eventUrl`:

```json
{
  "start_time": "2020-01-01T12:00:00Z",
  "recording_url": "https://api.nexmo.com/v1/files/aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "size": 12345,
  "recording_uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "end_time": "2020-01-01T12:01:00Z",
  "conversation_uuid": "bbbbbbbb-cccc-dddd-eeee-0123456789ab",
  "timestamp": "2020-01-01T14:00:00.000Z"
}
```

Possible return parameters are:

 Name | Description
 -- | --
 `recording_uuid` | The unique ID for the Call. <br>**Note**: `recording_uuid` is not the same as the file uuid in *recording_url*.
 `recording_url` | The  URL to the file containing the Call recording.
 `start_time`  | The time the recording started in ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ`. For example `2020-01-01T12:00:00Z`.
 `end_time`  | The time the recording finished in ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ`. For example `2020-01-01T12:00:00Z`.
 `size` | The size of the recording at *recording_url* in bytes. For example: `603423`
 `conversation_uuid` | The unique ID for this Call.

## Conversation

You can use the `conversation` action to create standard or moderated conferences, while preserving the communication context. Using `conversation` with the same `name` reuses the same persisted [Conversation](/conversation/concepts/conversation). The first person to call the virtual number assigned to the conversation creates it. This action is synchronous.

> **Note**: you can invite up to 50 people to your Conversation.

The following NCCO examples show how to configure different types of Conversation. You can use the `answer_url` webhook GET request parameters to ensure you deliver one NCCO to participants and another to the moderator.

```tabbed_content
source: '/_examples/voice/guides/ncco-reference/conversation'
```

You can use the following options to control a *conversation* action:

Option | Description | Required
-- | -- | --
`name` | The name of the Conversation room. Names are namespaced to the application level. | Yes
`musicOnHoldUrl` | A URL to the *mp3* file to stream to participants until the conversation starts. By default the conversation starts when the first person calls the virtual number associated with your Voice app. To stream this mp3 before the moderator joins the conversation, set *startOnEnter* to *false* for all users other than the moderator. | No
`startOnEnter` | The default value of *true* ensures that the conversation starts when this caller joins conversation `name`. Set to *false* for attendees in a moderated conversation. | No
`endOnExit` | Specifies whether a moderated conversation ends when the moderator hangs up. This is set to *false* by default, which means that the conversation only ends when the last remaining participant hangs up, regardless of whether the moderator is still on the call. Set `endOnExit` to *true* to terminate the conversation when the moderator hangs up. | No
`record` | Set to *true* to record this conversation. For standard conversations, recordings start when one or more attendees connects to the conversation. For moderated conversations, recordings start when the moderator joins. That is, when an NCCO is executed for the named conversation where *startOnEnter* is set to *true*. When the recording is terminated, the URL you download the recording from is sent to the event URL. You can override the default recording event URL and default HTTP method by providing custom `eventUrl` and `eventMethod` options in the `conversation` action definition. <br>By default audio is recorded in MP3 format. See the [recording](/voice/voice-api/guides/recording#file-formats) guide for more details. | No
`canSpeak` | A list of leg UUIDs that this participant can be heard by. If not provided, the participant can be heard by everyone. If an empty list is provided, the participant will not be heard by anyone | No
`canHear` | A list of leg UUIDs that this participant can hear. If not provided, the participant can hear everyone. If an empty list is provided, the participant will not hear any other participants| No
`mute` | Set to *true* to mute the participant. The audio from the participant will not be played to the conversation and will not be recorded. When using `canSpeak`, the `mute` parameter is not supported. | No

## Connect

You can use the `connect` action to connect a call to endpoints such as phone numbers or a VBC extension.

This action is synchronous, after a *connect* the next action in the NCCO stack is processed. A connect action ends when the endpoint you are calling is busy or unavailable. You ring endpoints sequentially by nesting connect actions.

The following NCCO examples show how to configure different types of connections.

```tabbed_content
source: '/_examples/voice/guides/ncco-reference/connect'
```

You can use the following options to control a `connect` action:

Option | Description | Required
-- | -- | --
`endpoint` | Array of `endpoint` objects to connect to. Currently supports a **maximum** of one `endpoint` object. [Available endpoint types](#endpoint-types-and-values). | Yes
`from` | A number in [E.164](https://en.wikipedia.org/wiki/E.164) format that identifies the caller.§§ This must be one of your Vonage virtual numbers, another value will result in the caller ID being unknown. If the caller is an app user, this option should be omitted.| No
`randomFromNumber` | Set to `true` to use a  random phone number as `from`. The number will be selected from the list of the numbers assigned to the current application. `randomFromNumber: true` cannot be used together with `from`. The default value is `false`. | No
`eventType` | Set to `synchronous` to: <ul class="Vlt-list Vlt-list--simple"><li>make the `connect` action synchronous</li><li>enable `eventUrl` to return an NCCO that overrides the current NCCO when a call moves to specific states.</li></ul> | No
`timeout` |  If the call is unanswered, set the number in seconds before Vonage stops ringing `endpoint`. The default value is `60`.
`limit` | Maximum length of the call in seconds. The default and maximum value is `7200` seconds (2 hours). | No
`machineDetection` | Configure the behavior when Vonage detects that a destination is an answerphone. Set to either: <ul class="Vlt-list Vlt-list--simple"><li>`continue` - Vonage sends an HTTP request to `event_url` with the Call event `machine`</li><li>`hangup` - end the Call</li></ul>   | No
`eventUrl` | Set the webhook endpoint that Vonage calls asynchronously on each of the possible [Call States](/voice/voice-api/guides/call-flow#call-states). If `eventType` is set to `synchronous` the `eventUrl` can return an NCCO that overrides the current NCCO when a timeout occurs. | No
`eventMethod` | The HTTP method Vonage uses to make the request to <i>eventUrl</i>. The default value is `POST`. | No
`ringbackTone` | A URL value that points to a `ringbackTone` to be played back on repeat to the **caller**, so they don't hear silence. The `ringbackTone` will automatically stop playing when the call is fully connected. It's not recommended to use this parameter when connecting to a phone endpoint, as the carrier will supply their own `ringbackTone`. Example: `"ringbackTone": "http://example.com/ringbackTone.wav"`. | No

### Endpoint Types and Values

#### Phone (PSTN) - phone numbers in E.164 format

Value | Description
-- | --
`type` | The endpoint type: `phone` for a PSTN endpoint.
`number` | The phone number to connect to in [E.164](https://en.wikipedia.org/wiki/E.164) format.
`dtmfAnswer` | Set the digits that are sent to the user as soon as the Call is answered. The `*` and `#` digits are respected. You create pauses using `p`. Each pause is 500ms.
`onAnswer` | A JSON object containing a required `url` key. The URL serves an NCCO to execute in the number being connected to, before that call is joined to your existing conversation. Optionally, the `ringbackTone` key can be specified with a URL value that points to a `ringbackTone` to be played back on repeat to the **caller**, so they do not hear just silence. The `ringbackTone` will automatically stop playing when the call is fully connected. Example: `{"url":"https://example.com/answer", "ringbackTone":"http://example.com/ringbackTone.wav" }`. Please note, the key `ringback` is still supported.

#### App - Connect the call to a RTC capable application

Value | Description
-- | --
`type` | The endpoint type: `app` for an [application](/client-sdk/setup/create-your-application).
`user` | The username of the user to connect to. This username must have been [added as a user](/api/conversation#createUser).

#### WebSocket - the WebSocket to connect to

Value | Description
-- | --
`type` | The endpoint type: `websocket` for a WebSocket.
`uri` | The URI to the websocket you are streaming to.
`content-type` | the internet media type for the audio you are streaming. Possible values are: `audio/l16;rate=16000` or `audio/l16;rate=8000`.
`headers` | a JSON object containing any metadata you want. See [connecting to a websocket](/voice/voice-api/guides/websockets#connecting-to-a-websocket) for example headers.

#### SIP - the SIP endpoint to connect to

Value | Description
-- | --
`type` | The endpoint type: `sip` for SIP.
`uri` | The SIP URI to the endpoint you are connecting to in the format `sip:rebekka@sip.example.com`. To use [TLS and/or SRTP](/voice/sip/overview#protocols), include respectively `transport=tls` or `media=srtp` to the URL with the semicolon `;` as a delimiter, for example: `sip:rebekka@sip.example.com;transport=tls;media=srtp`. 
`headers` | `key` => `value` string pairs containing any metadata you need e.g. `{ "location": "New York City", "occupation": "developer" }`. The headers are transmitted as part of the SIP INVITE as `X-key: value` headers. So in the example, these headers are sent: `X-location: New York City` and `X-occupation: developer`. 

> To understand how your application can receive and handle SIP Custom Headers instead, check the following page on [Programmable SIP](/voice/sip/concepts/programmable-sip#custom-sip-headers)

#### VBC - the Vonage Business Cloud (VBC) extension to connect to

Value | Description
-- | --
`type` | The endpoint type: `vbc` for a VBC extension.
`extension` | the VBC extension to connect the call to.

## Talk

The `talk` action sends synthesized speech to a Conversation.

The text provided in the talk action can either be plain, or formatted using [SSML](/voice/voice-api/guides/customizing-tts). SSML tags provide further instructions to the text-to-speech synthesizer which allow you to set pitch, pronunciation and to combine together text in multiple languages. SSML tags are XML-based and sent inline in the JSON string.

By default, the talk action is synchronous. However, if you set *bargeIn* to *true* you must set an *input* action later in the NCCO stack.
The following NCCO examples shows how to send a synthesized speech message to a Conversation or Call:

```tabbed_content
source: '/_examples/voice/guides/ncco-reference/talk'
```

You can use the following options to control a *talk* action:

| Option | Description | Required |
| -- | -- | -- |
| `text` | A string of up to 1,500 characters (excluding SSML tags) containing the message to be synthesized in the Call or Conversation. A single comma in `text` adds a short pause to the synthesized speech. To add a longer pause a `break` tag needs to be used in SSML. To use [SSML](/voice/voice-api/guides/customizing-tts) tags, you must enclose the text in a `speak` element. | Yes |
| `bargeIn` | Set to `true` so this action is terminated when the user interacts with the application either with DTMF or ASR voice input. Use this feature to enable users to choose an option without having to listen to the whole message in your <a href="/voice/voice-api/guides/interactive-voice-response">Interactive Voice Response (IVR)</a>. If you set `bargeIn` to `true` the next non-talk action in the NCCO stack <b>must</b> be an `input` action. The default value is `false`. <br /><br />Once `bargeIn` is set to `true` it will stay `true` (even if `bargeIn: false` is set in a following action) until an `input` action is encountered | No |
| `loop` | The number of times `text` is repeated before the Call is closed. The default value is 1. Set to 0 to loop infinitely. | No |
| `level` | The volume level that the speech is played. This can be any value between `-1` to `1` with `0` being the default.  | No |
| `language` | The language ([BCP-47](https://tools.ietf.org/html/bcp47) format) for the message you are sending. Default: `en-US`. Possible values are listed in the [Text-To-Speech guide](/voice/voice-api/guides/text-to-speech#supported-languages). | No |
| `style` | The vocal style (vocal range, tessitura and timbre). Default: `0`. Possible values are listed in the [Text-To-Speech guide](/voice/voice-api/guides/text-to-speech#supported-languages). | No |

## Stream
The `stream` action allows you to send an audio stream to a Conversation

By default, the stream action is synchronous. However, if you set *bargeIn* to *true* you must set an *input* action later in the NCCO stack.

The following NCCO example shows how to send an audio stream to a Conversation or Call:

```tabbed_content
source: '/_examples/voice/guides/ncco-reference/stream'
```

You can use the following options to control a *stream* action:

Option | Description | Required
-- | -- | --
`streamUrl` | An array containing a single URL to an mp3 or wav (16-bit) audio file to stream to the Call or Conversation. | Yes
`level` |  Set the audio level of the stream in the range -1 >=level<=1 with a precision of 0.1. The default value is *0*. | No
`bargeIn` | Set to `true` so this action is terminated when the user interacts with the application either with DTMF or ASR voice input. Use this feature to enable users to choose an option without having to listen to the whole message in your [Interactive Voice Response (IVR](/voice/guides/interactive-voice-response) ). If you set `bargeIn` to `true` on one more Stream actions then the next non-stream action in the NCCO stack **must** be an `input` action. The default value is `false`.<br /><br />Once `bargeIn` is set to `true` it will stay `true` (even if `bargeIn: false` is set in a following action) until an `input` action is encountered. | No
`loop` | The number of times `audio` is repeated before the Call is closed. The default value is `1`. Set to `0` to loop infinitely. | No

The audio stream referred to should be a file in MP3 or WAV format. If you have issues with the file playing, please encode it to the following technical specification: [What kind of prerecorded audio files can I use?](https://help.nexmo.com/hc/en-us/articles/115007447567)

> If you play the same audio file multiple times, for example using the same recording in many calls, consider adding a [Cache-Control](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) header to the URL response with proper values.
>
> ```
> Cache-Control: public, max-age=360000
> ```
>
> This allows Vonage to cache your audio file instead of downloading it each time, which may significantly improve performance and user experience. Caching is supported for both HTTP and HTTPS URLs.

## Input

You can use the `input` action to collect digits or speech input by the person you are calling. This action is synchronous, Vonage processes the input and forwards it in the [parameters](#input-return-parameters) sent to the `eventUrl` webhook endpoint you configure in your request. Your webhook endpoint should return another NCCO that replaces the existing NCCO and controls the Call based on the user input. You could use this functionality to create an Interactive Voice Response (IVR). For example, if your user presses *4* or says "Sales", you return a [connect](#connect) NCCO that forwards the call to your sales department.

The following NCCO example shows how to configure an IVR endpoint:

```json
[
  {
    "action": "talk",
    "text": "Please enter a digit or say something"
  },
  {
    "action": "input",
    "eventUrl": [
      "https://example.com/ivr"
    ],
    "type": [ "dtmf", "speech" ],
    "dtmf": {
      "maxDigits": 1
    },
    "speech": {
      "context": [ "sales", "support" ]
    }
  }
]
```

The following NCCO example shows how to use `bargeIn` to allow a user to interrupt a `talk` action. Note that an `input` action **must** follow any action that has a `bargeIn` property (e.g. `talk` or `stream`).

```json
[
  {
    "action": "talk",
    "text": "Please enter a digit or say something",
    "bargeIn": true
  },
  {
    "action": "input",
    "eventUrl": [
      "https://example.com/ivr"
    ],
    "type": [ "dtmf", "speech" ],	
    "dtmf": {
      "maxDigits": 1
    },
    "speech": {
      "context": [ "sales", "support" ]
    }	
  }
]
```

The following options can be used to control an `input` action:

Option | Description | Required
-- | -- | --
`type` | Acceptable input type, can be set as `[ "dtmf" ]` for DTMF input only, `[ "speech" ]` for ASR only, or `[ "dtmf", "speech" ]` for both. | Yes
`dtmf` | [DTMF settings](#dtmf-input-settings). | No
`speech` | [Speech recognition settings](#speech-recognition-settings). | No
`eventUrl` | Vonage sends the digits pressed by the callee to this URL 1) after `timeOut` pause in activity or when *#* is pressed for DTMF or 2) after user stops speaking or `30` seconds of speech for speech input.  | No
`eventMethod` | The HTTP method used to send event information to `event_url` The default value is `POST`.| No

#### DTMF Input Settings

Option | Description | Required
-- | -- | --
`timeOut` | The result of the callee's activity is sent to the `eventUrl` webhook endpoint `timeOut` seconds after the last action. The default value is *3*. Max is 10.| No
`maxDigits` | The number of digits the user can press. The maximum value is `20`, the default is `4` digits. | No
`submitOnHash` | Set to `true` so the callee's activity is sent to your webhook endpoint at `eventUrl` after they press *#*. If *#* is not pressed the result is submitted after `timeOut` seconds. The default value is `false`. That is, the result is sent to your webhook endpoint after `timeOut` seconds. | No

#### Speech Recognition Settings

Option | Description | Required
-- | -- | --
`uuid` | The unique ID of the Call leg for the user to capture the speech of, defined as an array with a single element. The first joined leg of the call by default. | No
`endOnSilence` | Controls how long the system will wait after user stops speaking to decide the input is completed. The default value is `2.0` (seconds). The range of possible values is between `0.4` seconds and `10.0` seconds. | No
`language` | Expected language of the user's speech. Format: BCP-47. Default: `en-US`. [List of supported languages](/voice/voice-api/guides/asr#language). | No
`context` | Array of hints (strings) to improve recognition quality if certain words are expected from the user. | No
`startTimeout` | Controls how long the system will wait for the user to start speaking. The range of possible values is between 1 second and 60 seconds. | No
`maxDuration` | Controls maximum speech duration (from the moment user starts speaking). The default value is 60 (seconds). The range of possible values is between 1 and 60 seconds. | No
`saveAudio` | Set to `true` so the speech input recording (`recording_url`) is sent to your webhook endpoint at `eventUrl`. The default value is `false`. | No


The following example shows the parameters sent to the `eventUrl` webhook for DTMF input:

```json
{
  "speech": { "results": [ ] },
  "dtmf": {
    "digits": "1234",
    "timed_out": true
  },
  "from": "15551234567",
  "to": "15557654321",
  "uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "conversation_uuid": "bbbbbbbb-cccc-dddd-eeee-0123456789ab",
  "timestamp": "2020-01-01T14:00:00.000Z"
}
```

The following example shows the parameters sent back to the `eventUrl` webhook for speech input:

```json
{
  "speech": {
    "recording_url": "https://api-us.nexmo.com/v1/files/eeeeeee-ffff-0123-4567-0123456789ab",
    "timeout_reason": "end_on_silence_timeout",
    "results": [
      {
        "confidence": "0.9405097",
        "text": "sales"
      },
      {
        "confidence": "0.70543784",
        "text": "sails"
      },
      {
        "confidence": "0.5949854",
        "text": "sale"
      }
    ]
  },
  "dtmf": {
    "digits": null,
    "timed_out": false
  },
  "from": "15551234567",
  "to": "15557654321",  
  "uuid": "aaaaaaaa-bbbb-cccc-dddd-0123456789ab",
  "conversation_uuid": "bbbbbbbb-cccc-dddd-eeee-0123456789ab",
  "timestamp": "2020-01-01T14:00:00.000Z"
}
```

### Input Return Parameters

See [Webhook Reference](/voice/voice-api/webhook-reference#input) for input parameters which are returned to the `eventUrl`.

## Notify

Use the `notify` action to send a custom payload to your event URL. Your webhook endpoint can return another NCCO that replaces the existing NCCO or return an empty payload meaning the existing NCCO will continue to execute.

```json
[
  {
    "action": "notify",
    "payload": {
      "foo": "bar"
    },
    "eventUrl": [
      "https://example.com/webhooks/event"
    ],
    "eventMethod": "POST"
  }
]
```

Option | Description | Required
-- | -- | --
`payload` | The JSON body to send to your event URL. | Yes
`eventUrl` | The URL to send events to. If you return an NCCO when you receive a notification, it will replace the current NCCO. | Yes
`eventMethod` | The HTTP method to use when sending `payload` to your `eventUrl`. | No

## Pay

The `pay` action collects credit card information with DTMF input in a secure ([PCI-DSS compliant](https://www.pcisecuritystandards.org/pci_security/)) way. See [Payments over the Phone](/voice/voice-api/guides/payments) guide to learn more.

```json
[
  {
    "action": "talk",
    "text": "Thank you for your order, we will charge 9 dollars 99 cents now"
  },
  {
    "action": "pay",
    "eventUrl": [
      "https://example.com/pay"
    ],
    "amount": 9.99
  }
]
```

The following options can be used to control an `pay` action:

Option | Description | Required
-- | -- | --
`amount` | Charge amount in US dollars, decimal value > `0`. | Yes
`eventUrl` | The URL to the webhook endpoint that is called asynchronously when payment is finished. | No
`prompts` | Array of [Prompt settings](#prompts-text-settings) | No
`voice` | [Text to speech voice settings](#prompts-voice-settings) | No

### Prompts Voice Settings
Option | Description | Required
-- | -- | --
`language` | The language ([BCP-47](https://tools.ietf.org/html/bcp47) format) for the prompts. Default: `en-US`. Possible values are listed in the [Text-To-Speech guide](/voice/voice-api/guides/text-to-speech#supported-languages). | No 
`style` | The vocal style (vocal range, tessitura and timbre). Default: `0`. Possible values are listed in the [Text-To-Speech guide](/voice/voice-api/guides/text-to-speech#supported-languages). | No 

### Prompts Text Settings
Option | Description | Required
-- | -- | --
`type` | Prompt type. Possible values: `CardNumber`, `ExpirationDate`, `SecurityCode` | Yes
`text` | Prompt text, for example, "Enter your card number". | Yes 
`errors` | [Error prompts settings](#error-prompts). | Yes

### Error Prompts
Possible errors depend on prompt `type` (see above):
Prompt Type | Error Type | Description 
-- | -- | --
`CardNumber` | `InvalidCardType` | The type of the card entered is not in the list of allowed card types
`CardNumber`| `InvalidCardNumber` | The card number entered is not correct
`ExpirationDate` | `InvalidExpirationDate` | Invalid month number or date in the past
`SecurityCode` | `InvalidSecurityCode` | Security code validation did not pass
`CardNumber`, `ExpirationDate`, `Security` | `Timeout` | No user input during 10 seconds

Option | Description | Required
-- | -- | --
`text` | Error prompt text, for example, "Invalid credit card number. Please try again". | Yes 

Error prompts example:

```json
{
  "type": "ExpirationDate",
  "text": "Please enter expiration date",
  "errors": {
    "InvalidExpirationDate": {
      "text": "Invalid expiration date. Please try again"
    },
    "Timeout": {
      "text": "Please enter your 4 digit credit card expiration date"
    }
  }
}
```