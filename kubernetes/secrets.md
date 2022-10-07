# Secrets

## ERROR: new line at the end

{% hint style="warning" %}
Below code will add `new line` at the end, which is unexpected for the most secrets
{% endhint %}

```
echo 123 | base64
MTIzCg==
```

If that secret will be used to start MySQL server. Server will drop strange message complaining to unknown parameter. Reason for that is `\n` move caret to a new line and new line should have separate command.&#x20;

The right way to generate secrets is

```
echo -n 123 | base64
```

