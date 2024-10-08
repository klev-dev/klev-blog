[//]: # ({"title": "Filters", "date": "2024-10-08"})

Filters
===============

Recently, we added support for log [filters](https://klev.dev/docs#filters) to [klev](https://klev.dev). They enable you to republish a subset of the messages to a different log, based on certain criteria.

To configure a filter, you'll need a source and a target logs, and a [CEL expression](https://cel.dev/) that evaluates to a boolean value. The CEL expression will be evaluated for each new message on the source log and if it evaluates to `true` the message will be copied to the target log, while preserving its `time`, `key` and `value` attributes.

Using filters
-------------

Lets look at an example of how you can use filters with [klev](https://klev.dev). As a setup, lets imagine we have 2 devices, pushing status messages to klev at the same log. We would like to see the latest status of each device, without having to search through the whole log. We could do this by using a couple of filters to extract only statuses for a certain device.

klev configuration
------------------

Start by creating the logs that we'll use:
```bash
$ klev-cli logs create --metadata "status for all devices" --trim-seconds 86400
{
  "log_id": "log_2n9on3qSjxy3qm2JfqdNrYbI5M0",
  "metadata": "status for all devices",
  "compacting": false,
  "trim_seconds": 86400
}

$ klev-cli logs create --metadata "status for device1" --trim-seconds 86400
{
  "log_id": "log_2n9oruCOUYlIn9inkOMRjCc2Ro2",
  "metadata": "status for device1",
  "compacting": false,
  "trim_seconds": 86400
}

$ klev-cli logs create --metadata "status for device2" --trim-seconds 86400
{
  "log_id": "log_2n9ouTERsIytcIjqM8RTZ4VJK9x",
  "metadata": "status for device2",
  "compacting": false,
  "trim_seconds": 86400
}
```

Next, lets define our filters. Assuming each device sends messages with a value in a JSON format `{"device_id": "XXX", ...}`, we can add the following filters:
```bash
$ klev-cli filters create --source-id log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --target-id log_2n9oruCOUYlIn9inkOMRjCc2Ro2 --expression 'toJson(value).device_id == "device1"' --metadata "all to device1"
{
  "filter_id": "trf_2nAibtSA99A5Iv89xIryywX2YPj",
  "metadata": "all to device1",
  "source_id": "log_2n9on3qSjxy3qm2JfqdNrYbI5M0",
  "target_id": "log_2n9oruCOUYlIn9inkOMRjCc2Ro2",
  "expression": "toJson(value).device_id == \"device1\""
}

$ klev-cli filters create --source-id log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --target-id log_2n9ouTERsIytcIjqM8RTZ4VJK9x --expression 'toJson(value).device_id == "device2"' --metadata "all to device2"
{
  "filter_id": "trf_2nAiggvwdQ7tIdfSdpWC3uQrTPG",
  "metadata": "all to device2",
  "source_id": "log_2n9on3qSjxy3qm2JfqdNrYbI5M0",
  "target_id": "log_2n9ouTERsIytcIjqM8RTZ4VJK9x",
  "expression": "toJson(value).device_id == \"device2\""
}
```

klev messages
------------------

Now, lets send some messages to the shared devices log. We'll publish inactive/active messages for each device:
```bash
$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device1", "status": "inactive"}'
{
  "next_offset": 1
}

$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device2", "status": "inactive"}'
{
  "next_offset": 2
}

$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device3", "status": "inactive"}'
{
  "next_offset": 3
}

$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device1", "status": "active"}'
{
  "next_offset": 4
}

$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device2", "status": "active"}'
{
  "next_offset": 5
}

$ klev-cli publish log_2n9on3qSjxy3qm2JfqdNrYbI5M0 --value '{"device_id": "device3", "status": "active"}'
{
  "next_offset": 6
}
```

Now lets get the latest message on the device specific logs:
```bash
$ klev-cli get-by-offset log_2n9oruCOUYlIn9inkOMRjCc2Ro2
{
  "offset": 1,
  "time": 1728422504051183,
  "value": "{\"device_id\": \"device1\", \"status\": \"active\"}"
}

$ klev-cli get-by-offset log_2n9ouTERsIytcIjqM8RTZ4VJK9x
{
  "offset": 1,
  "time": 1728422508017241,
  "value": "{\"device_id\": \"device2\", \"status\": \"active\"}"
}
```

Conclusion
----------

By using the power of [klev](https://klev.dev), we can conditionally redirect messages from log to another, enabling out application to see only a subset of the messages, as needed.

&#128075; Enjoy hacking!