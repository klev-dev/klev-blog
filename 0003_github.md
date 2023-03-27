[//]: # ({"title": "Webhooks and GitHub", "date": "2023-03-27"})

Webhooks
========

klev has support for ingesting via webhooks. We support Slack, Stripe, and since recently GitHub.

You configure a webhook with provider, secret and a target log. Once klev receives the http call, it will validate it, and then push a new message on the target log with the webhook payload as a value. 

GitHub Webhooks
---------------

Let's try to add a GitHub webhook. We'll need to first configure klev, then add a webhook to GitHub, and finally see the payloads in klev.

klev configuration
------------------

Start by adding a log, that will contain the payloads:
```bash
$ klev logs create --metadata "github webhooks"
{
  "log_id": "log_2NcGE3LLPDhHS2pPwUh946EibAK",
  "metadata": "github webhook",
  "compacting": false
}
```

Then add the webhook itself, like:
```bash
$ klev webhooks create --log-id log_2NcGE3LLPDhHS2pPwUh946EibAK --metadata "github webhook" --type "github" --secret REDACTED
{
  "webhook_id": "wh_2NcGTsqkKANRT9mxieRbSpKIlaB",
  "log_id": "log_2NcGE3LLPDhHS2pPwUh946EibAK",
  "metadata": "github webhook",
  "type": "github"
}
```

GitHub configuration
--------------------

Now create a new webhook in GitHub with the following parameters:
 - Payload URL: https://webhooks.klev.dev/wh_2NcGTsqkKANRT9mxieRbSpKIlaB
 - Content Type: `application/json` (xml works too)
 - Secret: REDACTED
 - leave everything else as a default

Now watch the status of the webhook in GitHub, and when it turns green go to the next step.

klev consume
------------

Now lets check what this webhook delivered us:

```bash
$ klev consume log_2NcGE3LLPDhHS2pPwUh946EibAK
{
  "next_offset": 1,
  "encoding": "string",
  "messages": [
    {
      "offset": 0,
      "time": 1679956862967113,
      "value": "{\"zen\":\"Anything added dilutes everything else.\"...",
    }
  ]
}
```

Conclusion
----------

By using the power of [klev](https://klev.dev), we ingested webhooks directly into klev, which we can then further process on as needed basis. 

&#128075; Enjoy hacking!
