---
title: "Session Expiration"
description: "Documentation on IdentityServer's session expiration feature, which automatically cleans up expired server-side sessions and can notify client applications via back-channel logout."
sidebar:
  order: 20
redirect_from:
  - /identityserver/v5/ui/server_side_sessions/session_expiration/
  - /identityserver/v6/ui/server_side_sessions/session_expiration/
  - /identityserver/v7/ui/server_side_sessions/session_expiration/
---

If the user session ends when the session cookie expires without explicitly triggering logout, there is most likely the
need to clean up the server-side session data.
In order to remove these expired records, there is an automatic cleanup mechanism that periodically scans for expired
sessions.
When these records are cleaned up, you can optionally notify the client that the session has ended via back-channel
logout.

## Expiration Configuration

The expiration configuration features can be configured with
the [server-side session options](/identityserver/reference/options#server-side-sessions).
It is enabled by default, but if you wish to disable it or change how often IdentityServer will check for expired
sessions, you can.

For example, to change the interval:

```cs
// Program.cs
builder.Services.AddIdentityServer(options => {
    options.ServerSideSessions.RemoveExpiredSessionsFrequency = TimeSpan.FromSeconds(60);
})
    .AddServerSideSessions();
```

To disable:

```cs
// Program.cs
builder.Services.AddIdentityServer(options => {
    options.ServerSideSessions.RemoveExpiredSessions = false;
})
    .AddServerSideSessions();
```

### Back-channel Logout

When the session cleanup job removes expired records, it will by default also
trigger [back-channel logout notifications](/identityserver/ui/logout/notification#back-channel-server-side-clients)
to client applications participating in the session. You can use this mechanism to create
an [inactivity timeout](/identityserver/ui/server-side-sessions/inactivity-timeout/) that applies across all your client applications.

The `ServerSideSessions.ExpiredSessionsTriggerBackchannelLogout` flag enables this behavior, and it is on by default.
