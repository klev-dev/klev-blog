[//]: # ({"title": "Egress webhooks", "date": "2023-09-23"})

Egress Webhooks
===============

We added support for egress webhooks to klev. Egress webhooks let you directly receive new messages that come to a log, without having to consume it directly. 

You configure an egress webhook with source log, destination url and the type of payload (e.g. if you want the full message or just the key or value of the message). klev will generate a secret for you, which you use to validate the incoming webhook.

Receiving webhooks
------------------

Lets look at an example of how you can receive webhooks from klev. We'll need to configure the log and the egress webhook, then listen for new messages that are coming in.

ngrok configuration
-------------------

To receive webhooks on the internet we need a public address. The easiest way to have one (that we know of) is by using [ngrok](https://ngrok.com). Go to their [dashboard](https://dashboard.ngrok.com), register for an account and start ngrok with:
```bash
$ ngrok http 9000
```

Then copy the `Forwarding` url, in this example https://7e087ef51c73.ngrok.app

klev configuration
------------------

Start by adding a log that will feed your webhook:
```bash
$ klev logs create --metadata "egress webhook test"
{
  "log_id": "log_2VnvpTOoNFQRf0G9peK1zKlfhlf",
  "metadata": "egress webhook test",
  "compacting": false
}
```

Then add the egress webhook itself:
```bash
$ klev egress-webhooks create --log-id "log_2VnvpTOoNFQRf0G9peK1zKlfhlf" --metadata "egress webhook test" --destination "https://7e087ef51c73.ngrok.app"
{
  "webhook_id": "ewh_2VnwJdcym3NLEaA6MUkrPWzKWkQ",
  "metadata": "egress webhook test",
  "log_id": "log_2VnvpTOoNFQRf0G9peK1zKlfhlf",
  "destination": "https://7e087ef51c73.ngrok.app",
  "payload": "message",
  "secret": "ewhsec_4WWS5n4YcVrX8FhtxTKG3ctFHud3amGsr"
}
```

klev receive
------------

Now we are ready to receive messages, so lets start it:
```bash
$ klev receive --secret "ewhsec_4WWS5n4YcVrX8FhtxTKG3ctFHud3amGsr"
running server at :9000
```

Now lets send a message from another terminal:
```bash
$ klev publish log_2VnvpTOoNFQRf0G9peK1zKlfhlf --value "hello world"
{
  "next_offset": 1
}
```

Back to the first terminal (where we run `klev receive`) you will see:
```bash
Offset: 0
 Time: 2023-09-23 17:42:07.24796 +0000 UTC
 Key: 
 Value: hello world
```

You can also check the status of the egress webhook by running:
```bash
$ klev egress-webhooks status "ewh_2Vo4QwZuZy1sK1zwbgrUOgwOaZB"
{
  "webhook_id": "ewh_2Vo4QwZuZy1sK1zwbgrUOgwOaZB",
  "active": true,
  "available_offset": 0,
  "next_offset": 1,
  "deliver_offset": 0,
  "deliver_time": 1695490928,
  "deliver_resp": "HTTP/1.1 200 OK\r\nContent-Length: 0\r\nDate: Sat, 23 Sep 2023 17:42:12 GMT\r\n\r\n",
  "next_deliver_time": 1695490928
}
``` 

Conclusion
----------

By using the power of [klev](https://klev.dev), we can receive messages from klev directly into our app, allowing us to be notified for new messages without consuming from klev all the time. 

&#128075; Enjoy hacking!
