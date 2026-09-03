# poor-mans-sendmail

Just a tiny implementation to test mail servers.

# Usage

## sendmail

Run `./sendmail` for a short description.

```
./sendmail <server> [<mail_from> [<rcpt_to> [mailfile]]]
```

`sendmail` calls `preprocess` internally to render the EML template before
sending. `FROM` and `TO` are passed automatically from the command-line
arguments; all other placeholders must be set via `--replace` if needed.

## preprocess

`preprocess` renders an EML template into a ready-to-send message. It is
called automatically by `sendmail`, but can also be used stand-alone:

```
./preprocess [options] <input.eml> [output.eml]
```

Without `output.eml` the result is written to stdout.

**Options**

| Option                 | Description                                                                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `--replace KEY=VALUE`  | Replace `INSERT-KEY-VARIABLE` with VALUE (repeatable). Keys are case-insensitive.                                                            |
| `--dkim-key TYPE=FILE` | Override the key file for algorithm family `rsa` or `ed25519` (repeatable). Default: `<selector>.key` in the same directory as `preprocess`. |

**Variable substitution**

Every `INSERT-KEY-VARIABLE` placeholder in the template is replaced.
`DATE` and `MESSAGEID` are filled automatically; any value can be overridden
with `--replace`. The script aborts if any placeholder remains unresolved
after substitution.

**DKIM signing**

If the template contains one or more `DKIM-Signature` headers, `preprocess`
signs them automatically. The current values of `bh=` and `b=` in the
template are ignored and always replaced with the computed values.

Key files are looked up by selector name (`s=` tag) in the same directory as
`preprocess` (e.g. selector `rsa-2026` → `rsa-2026.key`). The script aborts
if a required key file is missing. Use `--dkim-key` to override the path per
algorithm family:

```sh
# Simple mail (no DKIM)
./preprocess --replace from=a@example.com --replace to=b@example.com examples/valid.eml

# DKIM mail — keys rsa-2026.key and ed25519-2026.key must exist next to preprocess
./preprocess --replace from=sender@mgm-sp.team --replace to=rcpt@server examples/dkim-valid.eml

# DKIM mail with explicit key paths
./preprocess --replace from=sender@mgm-sp.team --replace to=rcpt@server \
             --dkim-key rsa=rsa-2026.key --dkim-key ed25519=ed25519-2026.key \
             examples/dkim-valid.eml

# Render to file, then verify
./preprocess --replace from=sender@mgm-sp.team --replace to=rcpt@server \
             examples/dkim-valid.eml /tmp/signed.eml
opendkim-testmsg < /tmp/signed.eml
```

# Rejection Test Cases

The following table lists all cases where a mail server with strict DMARC enforcement (`p=reject`) should reject a message.
DMARC passes if **either** SPF **or** DKIM authentication succeeds **and** the authenticated domain aligns with the `From` header domain.

| SPF      | MAIL FROM aligned with FROM | DKIM | DKIM `d=` aligned with FROM | Reason                 | Test |
| -------- | --------------------------- | ---- | --------------------------- | ---------------------- | ---- |
| hardfail | –                           | –    | –                           | SPF `-all`             | #4   |
| fail     | –                           | –    | –                           | DMARC fail, `p=reject` | #8   |
| fail     | –                           | fail | fail                        | DMARC fail, `p=reject` | #10  |
| pass     | no                          | fail | no                          | DMARC fail, `p=reject` | #11  |
| pass     | no                          | –    | –                           | DMARC fail, `p=reject` | #12  |
| pass     | no                          | pass | no                          | DMARC fail, `p=reject` | –    |
| fail     | –                           | pass | no                          | DMARC fail, `p=reject` | –    |

# Hints

Consider using the following command to set-up an smtpd your own.

python -m smtpd -n -d -c DebuggingServer 0.0.0.0:25
