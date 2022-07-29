# Email Consul integration

This HSL module can be used to send send email delivery results to Email Consul.

## Exported functions

### emailconsul_queue($id, $recipient, $sender, $connection, $jobid = none)

This function should be called when queueing the message in the [EOD](https://docs.halon.io/hsl/eod.html) script.

**Params**

- `$id` The id of the [queued](https://docs.halon.io/hsl/eodonce.html#EODMailMessage.queue) message
- `$recipient` The [recipient](https://docs.halon.io/hsl/eodonce.html#recipient) of the [queued](https://docs.halon.io/hsl/eodonce.html#EODMailMessage.queue) message
- `$sender` The [sender](https://docs.halon.io/hsl/eodonce.html#a6) of the [queued](https://docs.halon.io/hsl/eodonce.html#EODMailMessage.queue) message
- `$connection` The [pre-defined](https://docs.halon.io/hsl/eodonce.html#v-c6) `$connection` variable
- `$jobid` The job id of the [queued](https://docs.halon.io/hsl/eodonce.html#EODMailMessage.queue) message

**Returns**

The queue event as an associative array.

**Example**

```
import { emailconsul_queue } from "modules/emailconsul/emailconsul.hsl";

// Queue message for all recipients
foreach ($transaction["recipients"] as $recipient) {
    $id = $mail->queue($transaction["sender"], $recipient["address"], $recipient["transportid"]);
    $emailconsul = emailconsul_queue($id, $recipient, $transaction["sender"], $connection);
    http_bulk("emailconsul", json_encode($emailconsul));
}
```

### emailconsul_delivery($arguments, $message)

This function should be called after every delivery attempt in the [Post-delivery](https://docs.halon.io/hsl/postdelivery.html) script.

**Params**

- `$arguments` The [pre-defined](https://docs.halon.io/hsl/postdelivery.html#v-z1) `$arguments` variable
- `$message` The [pre-defined](https://docs.halon.io/hsl/postdelivery.html#v-m1) `$message` variable

**Returns**

The delivery event as an associative array.

**Example**

```
import { emailconsul_delivery } from "modules/emailconsul/emailconsul.hsl";

$emailconsul = emailconsul_delivery($arguments, $message);
http_bulk("emailconsul", json_encode($emailconsul));
```

### emailconsul_async($dsn, $recipient, $dsnrecipient)

This function should be called when receiving a DSN message in the [EOD](https://docs.halon.io/hsl/eod.html) script.

**Params**

- `$dsn` The [parsed](https://github.com/halon-extras/dsn) dsn message
- `$recipient` The [recipient](https://docs.halon.io/hsl/eodonce.html#recipient) of the message
- `$dsnrecipient` The recipient inside the [parsed](https://github.com/halon-extras/dsn) dsn message

**Returns**

The async delivery event as an associative array.

**Example**

```
import { dsn_parse } from "modules/dsn/dsn.hsl";
import { emailconsul_async } from "modules/emailconsul/emailconsul.hsl";

$dsn = dsn_parse($arguments["mail"]);
if ($dsn) {
    foreach ($transaction["recipients"] as $recipient) {
        foreach ($dsn["recipients"] as $dsnrecipient) {
            $emailconsul = emailconsul_async($dsn, $recipient, $dsnrecipient);
            http_bulk("emailconsul", json_encode($emailconsul));
        }
    }
}
```

### emailconsul_arf($arf)

This function should be called when receiving a ARF message in the [EOD](https://docs.halon.io/hsl/eod.html) script.

**Params**

- `$arf` The [parsed](https://github.com/halon-extras/arf) arf report

**Returns**

The feedback loop event as an associative array.

**Example**

```
import { arf_parse } from "modules/arf/arf.hsl";
import { emailconsul_arf } from "modules/emailconsul/emailconsul.hsl";

$arf = arf_parse($arguments["mail"]);
if ($arf) {
    $emailconsul = emailconsul_arf($arf);
    http_bulk("emailconsul", json_encode($emailconsul));
}
```

## Plugin configuration

This HSL module requires that the [http-bulk](https://github.com/halon-extras/http-bulk) plugin is installed.

```
plugins:
  - id: http-bulk
    config:
      queues:
        - id: emailconsul
          path: /var/run/emailconsul.jlog
          format: jsonarray
          url: https://api.dev.emailconsul.com/halon/<customerID>
          min_items: 500
          max_items: 500
          max_interval: 60
          headers:
            - "X-API-Key: <apiKey>"
```

You need to replace `<customerID>` and `<apiKey>` with your own values.