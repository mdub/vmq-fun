# vmq-fun

A test-rig for [VerneMQ](https://vernemq.com/).

I'm running `vernemq` in a Docker container, and poking at it via [ruby-mqtt](https://github.com/njh/ruby-mqtt).

Running `auto/dev` will start a "vernemq" container, then drop you into a interactive "dev" environment, suitable for running the test scripts.

`q` is the entrypoint.  Generate a message as follows:

    root@dev:/work# echo hello | q pub firehose

To generate _lots_ of messages, use the `timestamps` script:

    timestamps | q pub firehose

Then (in _another_ `auto/dev` window), subscribe to messages like this:

    root@dev:/work# q sub firehose

Enjoy!
