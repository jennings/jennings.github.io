---
title: So you need to issue API credentials
date: "2024-04-12T08:00:00-07:00"
layout: post
---

You host a popular web site or service and now you need to issue API
credentials.

Please, _please_, **PLEASE** keep these rules in mind when designing your
system.

This applies to any kind of shared secret between your API and its consumers.
You might call these "API keys", or if you use OAuth you probably call this a
"client secret".

## Summary

There are really only two ways you can implement API keys.

The first option is to always issue two keys and let your clients regenerate them individually:

<div style="border: solid 1px black; margin: 1em 2em; display: grid; grid-template-columns: min-content 1fr min-content min-content; gap: 3px;">
    <div>Primary key</div>
    <input type="text" readonly value="abcdefghijklmnop" />
    <button type="button">Regenerate</button>
    <div>Last used: 4 months ago (2024-01-20 17:34 UTC)</div>
    <div>Secondary key</div>
    <input type="text" readonly value="zyxwvutsrqponmlk" />
    <button type="button">Regenerate</button>
    <div>Last used: 10 minutes ago (2024-04-12 08:31 UTC)</div>
</div>

The second option is to allow your clients to create and revoke keys at will:

<div style="border: solid 1px black; margin: 1em 2em; display: grid; grid-template-columns: min-content 1fr min-content min-content; gap: 3px;">
    <div>Primary key</div>
    <input type="text" readonly value="abcdefghijklmnop" />
    <button type="button">Regenerate</button>
    <div>Last used: 4 months ago (2024-01-20 17:34 UTC)</div>
    <div>Secondary key</div>
    <input type="text" readonly value="zyxwvutsrqponmlk" />
    <button type="button">Regenerate</button>
    <div>Last used: 10 minutes ago (2024-04-12 08:31 UTC)</div>
</div>

## At least two valid keys at the same time

Each client should have at least **two** valid keys at the same time.

Sometimes your API consumers will need to rotate their API keys. Maybe they had
a breach, maybe they accidentally committed their key to a repository, or maybe
they just do it for the thrill.

If you only allow each client only has a single key, they must accept downtime
to rotate it and perform the whole process at once:

- Regenerate the key. This invalidates the previous key.
- Replace the key in their configuration.
- Reload configuration or restart their application.

Even if you accept revoked keys for a short grace period, this still forces
clients to do this whole process at once, which might be difficult to
coordinate.

If clients have multiple valid keys, then they can do this process with no downtime:

- Generate a new key. The old key remains valid and the client continues using
  it.
- During a maintenance window, replace the key in configuration.
- Verify the old key is no longer used anywhere.
- Revoke the old key.

## Regenerate keys separately

Keys must be individually revokable.

## Add permissions to keys without rotating

A key's permissions should be changable without issuing a new key.

Rotating a key can be an operational headache. A new version of the application
that needs new permissions should not require a key rotation.

## Show the last time when each key was used

Clients need to know when they've successfully rotated keys. Therefore, you
should show when the last time each key was used to authenticate.
