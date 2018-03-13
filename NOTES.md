# Experiments

## Scene 1: basic subscription

- `vernemq 0.13.1`
- clean install

`/var/lib/vernemq/msgstore/` starts at 532Kb

    $ docker-compose exec vernemq bash
    root@vernemq:/# du -s /var/lib/vernemq/msgstore/
    532	/var/lib/vernemq/msgstore/

I published 100 10-byte messages to topic `d/123` (with `retain=false`, `qos=1`)

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

`/var/lib/vernemq/msgstore/` did not grow!

    root@vernemq:/# du -s /var/lib/vernemq/msgstore/
    532	/var/lib/vernemq/msgstore/

I got client `bob` to subscribe to `d/+`

    root@dev:/work# bin/q -i bob sub d/+

then published another 100 messages (with `bob` online)

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

`bob` received all the messages. Surprisingly, although the messages were all received,
`/var/lib/vernemq/msgstore/` increased by ~50Kb!

    root@vernemq:/# du -s /var/lib/vernemq/msgstore/
    588	/var/lib/vernemq/msgstore/

Why does it do this?

I disconnected `bob`, and published another 100. `/var/lib/vernemq/msgstore/` did not grow. I guess this is because the subscription wasn't "durable".

Re-connecting `bob` and publishing more messages caused disk usage to increase again:

    root@dev:/work# yes 123456789 | head -100 | bin/q pub d/123

    root@vernemq:/# du -s /var/lib/vernemq/msgstore/
    628	/var/lib/vernemq/msgstore/

### Questions

- Why sending and successfully receiving messages consume disk space?
