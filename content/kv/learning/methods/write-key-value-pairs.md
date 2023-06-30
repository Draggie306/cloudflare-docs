---
pcx_content_type: concept
title: Write key-value pairs
weight: 7
---

# Write key-value pairs

To create a new key-value pair, or to update the value for a particular key, call the `put()` method on any namespace you have bound to your script. 

The basic form of this method looks like this:

```js
await NAMESPACE.put(key, value);
```

## Parameters

{{<definitions>}}

- `key` {{<type>}}string{{</type>}}

  - The key to associate with the value. A key cannot be empty, `.` or `..`. All other keys are valid. Keys have a maximum length of 512 bytes.

- `value` {{<type>}}string{{</type>}} | {{<type>}}ReadableStream{{</type>}} | {{<type>}}ArrayBuffer{{</type>}}
  - The value to store. The type is inferred.

{{</definitions>}}

This method returns a `Promise` that you should `await` on to verify a successful update.

The maximum size of a value is 25 MiB.

You can also [write key-value pairs from the command line with Wrangler](/workers/wrangler/workers-kv/) and [write data via the API](/api/operations/workers-kv-namespace-write-key-value-pair-with-metadata).

Due to the eventually consistent nature of Workers KV, concurrent writes can end up overwriting one another. It is a common pattern to write data from a single process via Wrangler or the API. This avoids competing, concurrent writes because of the single stream. All data is still readily available within all Workers bound to the namespace.

Writes are immediately visible to other requests in the same global network location, but can take up to 60 seconds to be visible in other parts of the world. 

Refer to [How KV works](/kv/learning/how-kv-works/) for more information on this topic.

## Write data in bulk

You can write more than one key-value pair at a time with Wrangler or [via the API](/api/operations/workers-kv-namespace-write-multiple-key-value-pairs). 

The bulk API can accept up to 10,000 KV pairs at once.

A `key` and `value` are required for each KV pair. The entire request size must be less than 100 megabytes. As of January 2022, Cloudflare does not support bulk writes from within a Worker.

## Expiring keys

Many common uses of Workers KV involve writing keys only meant to be valid for a certain amount of time. Rather than requiring applications to remember to delete such data at the appropriate time, Workers KV offers the ability to create keys that automatically expire. You may configure expiration to occur either at a particular point in time or after a certain amount of time has passed since the key was last modified.

Once the expiration time of an expiring key is reached, it will be deleted from the system. After its deletion, attempts to read the key will behave as if the key does not exist. The deleted key will not count against the namespace’s storage usage for billing purposes.

There are two ways to specify when a key should expire:

1.  Set a key's expiration using an absolute time specified in a number of [seconds since the UNIX epoch](https://en.wikipedia.org/wiki/Unix_time). For example, if you wanted a key to expire at 12:00AM UTC on April 1, 2019, you would set the key’s expiration to `1554076800`.

2.  Set a key's expiration time to live (TTL) using a relative number of seconds from the current time. For example, if you wanted a key to expire 10 minutes after creating it, you would set its expiration TTL to `600`.

Both of these options are usable when writing a key inside a Worker or when writing keys using the API.

As of January 2022, expiration targets that are less than 60 seconds into the future are not supported. This is true for both expiration methods.

## Create expiring keys

The `put()` method has an optional third parameter. 

It accepts an object with optional fields that allow you to customize the behavior of the `put()` method. You can set either `expiration` or `expirationTtl`, depending on how you want to specify the key’s expiration time. 

To do this, run one of the two commands below to set an expiration when writing a key from within a Worker:

{{<definitions>}}

- `NAMESPACE.put(key, value, {expiration: secondsSinceEpoch})` {{<type>}}Promise{{</type>}}

- `NAMESPACE.put(key, value, {expirationTtl: secondsFromNow})` {{<type>}}Promise{{</type>}}

{{</definitions>}}

These assume that `secondsSinceEpoch` and `secondsFromNow` are variables defined elsewhere in your Worker code.

You can also [write with an expiration on the command line via Wrangler](/workers/wrangler/workers-kv/) or [via the API](/api/operations/workers-kv-namespace-write-key-value-pair-with-metadata).

## Metadata

To associate some metadata with a key-value pair, set `metadata` to any arbitrary object (must serialize to JSON) in the `put()` options object on a `put()` call. 

To do this in a Worker:

```js
await NAMESPACE.put(key, value, {
  metadata: { someMetadataKey: "someMetadataValue" },
});
```

The serialized JSON representation of the metadata object must be no more than 1024 bytes in length.