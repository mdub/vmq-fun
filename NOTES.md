# Experiments

## Scene 1: basic subscription

- `vernemq 0.13.1`
- clean install

`/var/lib/vernemq/msgstore` starts at 532Kb

    $ docker-compose exec vernemq bash
    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    532	/var/lib/vernemq/msgstore

I published 100 10-byte messages to topic `d/123` (with `retain=false`, `qos=1`)

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

`/var/lib/vernemq/msgstore` did not grow!

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    532	/var/lib/vernemq/msgstore

I got client `bob` to subscribe to `d/+`

    root@dev:/work# bin/q -i bob sub d/+

then published another 100 messages (with `bob` online)

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

`bob` received all the messages. Surprisingly, although the messages were all received,
`/var/lib/vernemq/msgstore` increased by ~50Kb!

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    588	/var/lib/vernemq/msgstore

Why does it do this?

I disconnected `bob`, and published another 100. `/var/lib/vernemq/msgstore` did not grow. I guess this is because the subscription wasn't "durable".

Re-connecting `bob` and publishing more messages caused disk usage to increase again:

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    628	/var/lib/vernemq/msgstore

### Questions

- Why sending and successfully receiving messages consume disk space?

## Scene 2: durable subscription

- `vernemq 0.13.1`
- clean install

`/var/lib/vernemq/msgstore` starts at 532Kb, again.

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    532	/var/lib/vernemq/msgstore

This time, I create a durable subscription (with `clean_session = false`).

    root@dev:/work# bin/q -i bob sub --persistent d/+

Again, publishing messages caused disk to grow:

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    588	/var/lib/vernemq/msgstore

Again, I disconnected `bob`, and published more messages.

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

This time, disk usage increasing, despite `bob` being offline.

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    616	/var/lib/vernemq/msgstore

Reconnecting `bob` caused messages to be delivered, _and_ disk usage to grow.

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    628	/var/lib/vernemq/msgstore

### Questions

- Why does _consuming_ messages consume disk space?

## Scene 3: try upgrading

I tried upgrading VerneMQ to the latest version.

```diff
diff --git a/vernemq/Dockerfile b/vernemq/Dockerfile
index 15c18a1..f072d40 100644
--- a/vernemq/Dockerfile
+++ b/vernemq/Dockerfile
@@ -10,7 +10,7 @@ RUN apt-get update && apt-get install -y \
     jq \
     && rm -rf /var/lib/apt/lists/*

-ENV VERNEMQ_VERSION 0.13.1
+ENV VERNEMQ_VERSION 1.3.0

 # Defaults
 ENV DOCKER_VERNEMQ_KUBERNETES_NAMESPACE default
```

then repeated the "Scene 2 " experiment.

Initial disk usage was same as before.

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    532	/var/lib/vernemq/msgstore

I got `bob` to subscribe (durably) again, and published some messages.

    root@dev:/work# bin/q -i bob sub --persistent d/+

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

Disk usage grew.

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    600	/var/lib/vernemq/msgstore

I took `bob` offline, and published some more.

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

    root@vernemq:/# du -s /var/lib/vernemq/msgstore
    616	/var/lib/vernemq/msgstore

I reconnected `bob` again.  Message came through, disk usage went up.

### Conclusion

- A simple upgrade to the latest version of VerneMQ (`1.3.0`) won't help.
