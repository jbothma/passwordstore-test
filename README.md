Secrets store for X
===================

Manage these secrets using `pass` from [passwordstore.org](https://www.passwordstore.org/).

- **Readers:** Everyone who needs to read a given secret must have their GPG key
  ID in the root .gpg-id (defaults) or in the .gpg-id of that secret.
- **Writers:** Everyone who needs to update a given secret must import and sign
  the GPG public key of everyone who needs to read that secret.

## Listing available secrets

Just run `pass`

     $ pass
    Password Store
    ├── apps
    │   ├── someapp
    │   │   └── DJANGO_SECRET
    │   └── umibukela
    │       ├── AWS_SECRET_ACCESS_KEY
    │       ├── DJANGO_SECRET
    │       └── POSTGRES_PASSWORD


## Creating and storing a secret

Use `pass generate ...` - see help for details.

     $ pass generate apps/someapp/DJANGO_SECRET 10
    mkdir: created directory '/home/aaa/.password-store/apps/someapp'
    [master d3e0107] Add generated password for apps/someapp/DJANGO_SECRET.
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 apps/someapp/DJANGO_SECRET.gpg
    The generated password for apps/someapp/DJANGO_SECRET is:
    <|zMEs!MR9


## Reading a secret

Run `pass show ...path...` to print to command line or `pass show --clip ...path...` or `-c` to copy it to clipboard. Pass will clear it from the clipboard after 45 seconds by default.

     $ pass apps/someapp/DJANGO_SECRET
    <|zMEs!MR9


## Adding or removing a Reader to a specific directory

This is useful to grant read access to additional readers for a specific directory, but not to everything. Removing the user is the same operation - just omit their gpg-id from the command.

Use `pass init -p ...path... ...all..gpg-ids..that..should..read..this...`.

    $ pass init -p apps/someapp aaa@gmail.com bbb@gmail.com
    Password store initialized for aaa@gmail.com, bbb@gmail.com (apps/someapp)
    [master f12716d] Set GPG id to aaa@gmail.com, bbb@gmail.com (apps/someapp).
     1 file changed, 1 insertion(+)
    apps/someapp/AWS_SECRET_ACCESS_KEY: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    apps/someapp/DJANGO_SECRET: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    apps/someapp/POSTGRES_PASSWORD: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    [master c23fb5b] Reencrypt password store using new GPG id aaa@gmail.com, bbb@gmail.com (apps/someapp).
     3 files changed, 0 insertions(+), 0 deletions(-)
     rewrite apps/someapp/DJANGO_SECRET.gpg (100%)
     rewrite apps/someapp/POSTGRES_PASSWORD.gpg (100%)
     rewrite apps/someapp/AWS_SECRET_ACCESS_KEY.gpg (100%)

We can now check that the default is still only core readers

     $ cat .password-store/.gpg-id
    aaa@gmail.com

And that `someapp` has an additional reader

     $ cat .password-store/apps/someapp/.gpg-id
    aaa@gmail.com
    bbb@gmail.com


BEWARE
------

Pay attention to the output.

Here's an example where the passphrase was incorrect when re-encrypting one of three files, and correct the second time. The result was that only the last two files were encrypted properly. The first file was not updated.

     $ pass init -p apps/someapp aaa@gmail.com bbbb@gmail.com
    Password store initialized for aaa@gmail.com, bbbb@gmail.com (apps/someapp)
    [master f12716d] Set GPG id to aaa@gmail.com, bbbb@gmail.com (apps/someapp).
     1 file changed, 1 insertion(+)
    apps/someapp/AWS_SECRET_ACCESS_KEY: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    gpg: decryption failed: No secret key
    apps/someapp/DJANGO_SECRET: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    apps/someapp/POSTGRES_PASSWORD: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    [master c23fb5b] Reencrypt password store using new GPG id aaa@gmail.com, bbbb@gmail.com (apps/someapp).
     2 files changed, 0 insertions(+), 0 deletions(-)
     rewrite apps/someapp/DJANGO_SECRET.gpg (100%)
     rewrite apps/someapp/POSTGRES_PASSWORD.gpg (100%)


To fix it, in this case the init command could be rerun, this time giving the correct passphrase the first time. Only the first file needed to be re-encrypted so the other files didn't change.

     $ pass init -p apps/someapp aaa@gmail.com bbbb@gmail.com
    Password store initialized for aaa@gmail.com, bbbb@gmail.com (apps/someapp)
    apps/someapp/AWS_SECRET_ACCESS_KEY: reencrypting to 0549C9936556A5B6 5AFE80D12EC61626
    [master 237b2a2] Reencrypt password store using new GPG id aaa@gmail.com, bbbb@gmail.com (apps/someapp).
     1 file changed, 0 insertions(+), 0 deletions(-)
     rewrite apps/someapp/AWS_SECRET_ACCESS_KEY.gpg (100%)
