# Experiments

## Scene 1

- `vernemq 0.13.1`
- clean install

`/var/lib/vernemq/msgstore/` starts at 532Kb

    root@684c2f888c93:/# du -s /var/lib/vernemq/msgstore/
    532	/var/lib/vernemq/msgstore/

I published 1000 10-byte messages to topic `d/123` (with `retain=false`, `qos=1`)

    yes 123456789 | head -1000 | bin/q pub d/123

`/var/lib/vernemq/msgstore/` did not grow.

I got client `bob` to subscribe to `d/+`

    bin/q -i bob sub d/+

then published another 1000 messages

    yes 123456789 | head -1000 | bin/q pub d/123

`/var/lib/vernemq/msgstore/` increased by ~396Kb

    # du -s /var/lib/vernemq/msgstore/
    924	/var/lib/vernemq/msgstore/

I disconnected `bob`, and published another 1000.
