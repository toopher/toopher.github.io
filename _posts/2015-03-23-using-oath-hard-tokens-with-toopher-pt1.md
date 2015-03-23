---
layout: post
category: 
tags: [oath otp]
author: Drew
display_title: OATH Hardware Tokens and Toopher part 1: the API
---
{% include JB/setup %}

Here at Toopher, our vision of the future of Authentication is in large part
inspired by frustration with those old hardware OTP generator tokens you might
have hanging around on your keychain
([case in point](https://www.youtube.com/watch?v=nkmk7g6Vjvc&list=PLBi1ZsAf5AFivxUo1cin18d04zerDTfpy&t=269)).
That being said, hardware OTP generators are a mature, well-understood technology.
For users who are unwilling or unable to pair their mobile device with their
online identity, hardware tokens are a good option for stron authentication.

This will be a two-part post.  In part 1, I'm going to talk about what kind of
OTP generators we support, and give an overview of the REST API endpoints we've
added to the Toopher API.  Part 2 will show some changes to our
[toopher-python](https://github.com/toopher/toopher-python) client library that
simplify access to the new endpoints, and give some example code to help
developers support OTP hard tokens in their Toopher-enabled applications.

## First Things First

OTP Hard-token support is a premium feature, and so you must have a [Plus-level]
(https://dev.toopher.com/pricing/) requester to access the new endpoints.
Additionally, you will be responsible for the costs of acquiring the physical
hardware tokens and distributing them to your users.

## Supported Token Vendors

We support any OTP generator that is compliant with the
OATH [HOTP (event-based)](http://tools.ietf.org/html/rfc4226)
or [TOTP (time-based)](http://tools.ietf.org/html/rfc6238) One-Time Password specifications.
There are [many vendors to choose from](https://www.google.com/?gws_rd=ssl#q=oath+hardware+token),
so feel free to pick the one you like.  

Tokens that are limited to proprietary OTP generation algorithms are not supported.

# API Endpoints

## Token Provisioning

Endpoint: `POST /oath_otp_validator/provision`

Parameters:

*   `secret` - REQUIRED - Hex-encoded oath seed
*   `requester_specified_id` - RECOMMENDED - Arbitrary string used by requester to reference this particular token later.
*   `otp_type` - OPTIONAL.  One of `hotp` or `totp`.  Default: `totp`
*   `otp_digits` - OPTIONAL.  If `otp_type` is `totp`, this will default to `8`.  If `otp_type` is `hotp`, this defaults to `6`
*   `algorithm` - OPTIONAL.  Default: `sha1`
*   `totp_step_size` - OPTIONAL.  Only relevant if `otp_type` is `totp`.  Default is `30`

Returns: a JSON representation of the OATH OTP Validator.  Example:

```
{
    "otp_type": "totp",
    "totp_step_size": 30,
    "created": "2015-02-03 09:15:52",
    "modified": "2015-02-03 09:15:52",
    "otp_digits": 8,
    "algorithm": "sha1",
    "id": "4c6f30a5-f927-4eef-94e0-ec3348a088d9",
    "hotp_event_counter": 0,
    "requester_specified_id": "token1"
}
```

## Token Status

Two endpoints are available, depending on whether the token is referenced by the `requester_specified_id` string, or the Toopher-assigned UUID:

Endpoint 1: `POST /oath_otp_validator/<uuid>`

Endpoint 2: `POST /oath_otp_validator`

Parameters: 

*   `requester_specified_id` - The string identifier assigned to the token by the requester.  Only relevant for Endpoint 2.

Returns: a JSON representation of the OATH OTP Validator, similar to what is returned by the provisioning endpoint.

## Bulk Token Provisioning

Endpoint: `POST /oath_otp_validator/bulk_provision`

This endpoint takes a single POST parameter: `oath_otp_validators`, which is a JSON array of discrete sets of oath_otp_validator parameters.  Example:

```
[
  {
    "requester_specified_id":"token1",
    "secret":"3132333435363738393031323334353637383930"
  },
  {
    "requester_specified_id":"token2",
    "secret":"1234567890ABCDEF1234567890ABCDEF12345"
  }
]
```

Returns: a JSON object with a single key `oath_otp_validators` containing an array of created OATH OTP validators.  Example:

```
{
    "oath_otp_validators": [
        {
            "otp_type": "totp",
            "totp_step_size": 30,
            "created": "2015-02-03 09:17:38",
            "hotp_event_counter": 0,
            "modified": "2015-02-03 09:17:38",
            "otp_digits": 8,
            "algorithm": "sha1",
            "id": "ec940fe1-89a2-4cbf-a692-4a7d21572c35",
            "requester_specified_id": "token1"
        },
        {
            "otp_type": "totp",
            "totp_step_size": 30,
            "created": "2015-02-03 09:17:38",
            "hotp_event_counter": 0,
            "modified": "2015-02-03 09:17:38",
            "otp_digits": 8,
            "algorithm": "sha1",
            "id": "c0277d40-ffb3-4ec0-a812-0483f02be6ad",
            "requester_specified_id": "token2"
        }
    ]
}
```

## Associate an OATH OTP Validator with a particular user

Two endpoints are available to associate a token with a user, depending on whether you wish to reference the user by username, or by UUID.

Endpoint 1: `POST /users/<user_id>/oath_otp_validators/associate`

Parameters:

*   `id` - OPTIONAL - The Toopher-assigned UUID for the token.
*   `requester_specified_id` - OPTIONAL - the arbitrary string used by the requester to refer to the particular token

Requesters MUST specify either `id` or `requester_specified_id` to assign an existing token to a user.

Endpoint 2: `POST /users/oath_otp_validators/associate`

Parameters: as described for Endpoint 1, with the addition of:

*   `user_name` : The name of the user

Returns: a JSON representation of the OATH OTP validator that was associated with the user

Instead of using an existing OATH validator that was created through one of the provisioning endpoints, requesters may opt to include all of the provisioning parameters for a previously-unknown OATH validator in this user association API call.  This allows requesters to provision and associate a new token with a user with a single API call.

## Dissociate an OATH OTP Validator from a particular user

Two endpoints are available to dissociate a token from a user, depending on whether you wish to reference the user by username, or by UUID.

Endpoint 1: `POST /users/<user_id>/oath_otp_validators/dissociate`

Parameters:

*   `id` - OPTIONAL - The Toopher-assigned UUID for the token.
*   `requester_specified_id` - OPTIONAL - the arbitrary string used by the requester to refer to the particular token

Requesters MUST specify either `id` or `requester_specified_id` to identify the token to dissociate

Endpoint 2: `POST /users/oath_otp_validators/dissociate`

Parameters: as described for Endpoint 1, with the addition of:

*   `user_name` : The name of the user

Returns: Empty response on success (status=200)

## List all OATH OTP Validators currently associated with a user

Two endpoints are available to list the tokens associated with a user, depending on whether you wish to reference the user by username, or by UUID.

Endpoint 1: `GET /users/<user_id>/oath_otp_validators`

Endpoint 2: `GET /users/oath_otp_validators`

Parameter: `user_name` (only relevant for Endpoint 2)

Returns: a JSON object with a single key `oath_otp_validators`, containing an array of all validators for the user (similar format to the bulk provisioning endpoint)

## Re-Synchronize an OATH OTP Validator

During normal authentication, Toopher API allows Event-based tokens (HOTP) to be out-of-sync by up to 20 events, and Time-based tokens (TOTP) to be out-of-sync by up to two time-steps.  With each successful authentication, the API validator is re-synchronized, so the need for manual resynchronization should be rare.  If manual resynchronization is necessary, two endpoints are available, depending on whether you wish to reference the token by the `requester_specified_id` string, or the Toopher-assigned UUID:

Endpoint 1: `POST /oath_otp_validator/<uuid>/resync`

Parameters:

*   `otp_sequence` - a comma-separated list of AT LEAST TWO consecutive OTPs generated by the token.

Endpoint 2: `POST /oath_otp_validator/resync`

Parameters: the same as Endpoint 1, and additionally:

*   `requester_specified_id` - The string identifier assigned to the token by the requester

Returns: Empty response on success (Status=200)

HOTP tokens are synchronized by searching the first 10,000 events of the OTP sequence.
TOTP tokens are synchronized by searching +/- 1000 time-steps.

## Deactivate a previously-provisioned OATH OTP Validator

When deactivating a validator, that validator is automatically dissociated with all users, and is no longer available to associate with users in the future.  However, a reference to the OATH validator is maintained in Toopher's database.  If the same token is later re-provisioned (defined by having the same `secret` and `requester_specified_id`), then the previously deactivated token will be re-activated, preserving its HOTP event counter / TOTP drift tracker.

Two endpoints are available, depending on whether the token is referenced by the `requester_specified_id` string, or the Toopher-assigned UUID:

Endpoint 1: `POST /oath_otp_validator/<uuid>/deactivate`

Endpoint 2: `POST /oath_otp_validator/deactivate`

Parameters: 

*   `requester_specified_id` - The string identifier assigned to the token by the requester.  Only relevant for Endpoint 2.

Returns: Empty response on success (Status=200)

