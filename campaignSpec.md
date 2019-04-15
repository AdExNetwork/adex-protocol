## Overview

The `spec` field within each channel describes the campaign it's associated with.

`campaignSpec` refers to the format of describing ad campaigns.

If a channel is associated with a campaign (in practice, all channels created by the AdEx dapp are), it's on-chain `spec` field would be set to a sha256 hash of the JSON blob of the `channel.spec` field.

Within the validator stack, the `spec` is submitted as part of the POST `/channel` request.

### campaignSpec wrapper

Because the `campaignSpec` format needs to be able to evolve rapidly, we can use a wrapper that also contains the format version.

**Please note,** this wrapper format is not in use as of Q1 2019 (v4.0). If we decide to use it later, we can obsolete the `channel.spec` field and introduce another field which contains this wrpaper.

* `version`: a semver version of the format
* `body`: the `campaignSpec` body

Example: `{ "version": "1.0.0-beta",  "body": "..." }`

### campaignSpec format: v1.0.0-beta

**NOTE:** all monetary values are represented as a string that represents a decimal BigNumber in the channel asset unit (BigNumString)

* `validators`: an array of Validator objects; should always be 2 elements, first being the leader, second being the follower
* `maxPerImpression`: BigNumStr, a maximum payment per impression
* `minPerImpression`: BigNumStr, minimum payment offered per impression
* `targeting`: optional, an array of TargetingTag
* `limits`: Limits object
* `created`: Number, a millisecond timestamp of when the campaign was created
* `nonce`: BigNumStr, a random number to ensure the campaignSpec hash is unique
* `withdrawPeriodStart`: Number, a millisecond timestamp of when the campaign should enter a withdraw period (no longer accept any events other than `CHANNEL_CLOSE`); a sane value should be lower than `channel.validUntil * 1000` and higher than `created`; it is recommended to set this at least one month prior to `channel.validUntil * 1000`
* `adUnits`: optional, an array of [AdUnit](#Adunit)


#### Validator

* `addr`: string, the corresponding value in `channel.validators`
* `url`: string, a HTTPS URL to the validator's sentry
* `fee`: BigNumStr, the total fee that will be paid out to this validator when they distribute the whole remaining channel deposit

#### TargetingTag

* `tag`: string, arbitrary tag name
* `score`: number, from 0 to 100

**NOTE:** the SDK will use this by intersecting it with the user's `TargetingTag` array, multiplying the scores of all `TargetingTag`s with the same `tag`, and summing all the products. For example, if a certain `AdUnit` has `[{tag: 'location_US', score: 5}, { tag: 'location_UK', score: 8 }]`, and the user has `[{ tag: 'location_UK', score: 100 }]`, the end result will be 800.


#### Limits



#### AdUnit

##### Spec properties (added to [ipfs] and can NOT be modified) 

* `ipfs`: string, valid [ipfs] hash of spec props below
* `type`: string, the type of the ad unit; currently, possible values are: `legacy_250x250`, `legacy_468x60`, `legacy_336x280`, `legacy_728x90`, `legacy_120x600`, `legacy_160x600` see [IAB ad unit guidelines](https://www.soflaweb.com/standard-banner-sizes-iab-ad-unit-guidelines/) and `iab_flex_{adUnitName}` (see [IAB's new ad portfolio](https://www.iab.com/newadportfolio/) and [PDF](https://www.iab.com/wp-content/uploads/2017/08/IABNewAdPortfolio_FINAL_2017.pdf))
* `mediaUrl`: string, a URL to the resource (usually PNG); must use the `ipfs://` protocol, to guarantee data immutability
* `mediaMime`: string, MIME type of the media, possible values at the moment are: `image/jpeg`, `image/png`
* `targetUrl`: string, the advertised URL
* `targeting`: an array of [TargetingTag](TargetingTag), optional
* `tags`: an array of [TargetingTag](#TargetingTag), meant for discovery between publishers/advertisers
* `owner`: user address from the session
* `created`: number, UTC timestamp in milliseconds, used as nonce for escaping duplicated spec [ipfs] hashes

##### Non spec properties (not added to ipfs and CAN be modified)

* `title`: string, the name of the unit used in platform UI
* `description`: string, arbitrary text used in platform UI
* `archived`: boolean, user can change it - used for filtering in platform UI
* `modified`: number, UTC timestamp in milliseconds, changed every time modifiable property is changed

[ipfs]: https://ipfs.io/