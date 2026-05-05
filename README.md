# poor-mans-sendmail

Just a tiny implementation to test mail servers.

# Usage

Run ./sendmail for a short description

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
