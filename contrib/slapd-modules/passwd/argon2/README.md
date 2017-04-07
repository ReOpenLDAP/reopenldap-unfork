Argon2 ReOpenLDAP support
-------------------------

pw-argon2.c provides support for ARGON2 hashed passwords in OpenLDAP. For
instance, one could have the LDAP attribute:

`userPassword: {ARGON2}$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHQ$DKlexoEJUoZTmkAAC3SaMWk30El9/RvVhlqGo6afIng`

or:

`userPassword: {ARGON2}$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHRzYWx0$qOCkx9nMeFlaGOO4DUmPDgrlUbgMMuO9T1+vQCFuyzw`

Both hash the password "secret", the first using the salt "saltsalt", the second using the salt "saltsaltsalt"

Building
--------

1. Acquire libsodium <https://github.com/jedisct1/libsodium>, e.g:

    `$ apt install libsodium-dev`

2. Configure and build ReOpenLDAP.

    ```
    $ configue --enable-modules --enable-contrib ...
    $ make
    $ make install
    ```

3. Edit your slapd.conf (eg. /etc/ldap/slapd.conf), and add:

    `moduleload ...path/to/contrib-pw-argon2.so`


Configuring
-----------

The {ARGON2} password scheme should now be recognised.

You can also tell OpenLDAP to use one of this scheme when processing LDAP
Password Modify Extended Operations, thanks to the password-hash option in
slapd.conf:

`password-hash	{ARGON2}`


Testing
-------

A quick way to test whether it's working is to customize the rootdn and
rootpw in slapd.conf, eg:

```
rootdn          "cn=admin,dc=example,dc=com"

# This hashes the string 'secret', with a random salt
rootpw          {ARGON2}$argon2i$v=19$m=4096,t=3,p=1$uJyf0UfB25SQTfX7oCyK2w$U45DJqEFwD0yFaLvTVyACHLvGMwzNGf19dvzPR8XvGc
```

Then to test, run something like:

`ldapsearch -b "dc=example,dc=com" -D "cn=admin,dc=example,dc=com" -x -w secret`


-- Test hashes:

Test hashes can be generated with argon2 (get the reference implementation with the CLI tool here: <https://github.com/P-H-C/phc-winner-argon2/>):
```
$ echo -n "secret" | argon2 "saltsalt" -e
$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHQ$DKlexoEJUoZTmkAAC3SaMWk30El9/RvVhlqGo6afIng

$ echo -n "secret" | argon2 "saltsaltsalt" -e
$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHRzYWx0$qOCkx9nMeFlaGOO4DUmPDgrlUbgMMuO9T1+vQCFuyzw

$ echo -n "secretsecret" | argon2 "saltsalt" -e
$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHQ$U0Pd/wEsssZ9bHezDA8oxHnWe01xftykEy+7ehM2vic

$ echo -n "secretsecret" | argon2 "saltsaltsalt" -e
$argon2i$v=19$m=4096,t=3,p=1$c2FsdHNhbHRzYWx0$fkvoOwKgVtlX9ZDqcHFyyArBvqnAM0Igca8SScB4Jsc
```


Alternatively we could modify an existing user's password with
ldappasswd, and then test binding as that user:

```
$ ldappasswd -D "cn=admin,dc=example,dc=com" -x -W -S uid=jturner,ou=People,dc=example,dc=com
New password: secret
Re-enter new password: secret
Enter LDAP Password: <cn=admin's password>

$ ldapsearch -b "dc=example,dc=com" -D "uid=jturner,ou=People,dc=example,dc=com" -x -w secret
```

---
# License
Copyright 1992-2017 ReOpenLDAP AUTHORS: please see AUTHORS file.
All rights reserved.

This file is part of ReOpenLDAP.

Redistribution and use in source and binary forms, with or without
modification, are permitted only as authorized by the OpenLDAP
Public License.

A copy of this license is available in the file LICENSE in the
top-level directory of the distribution or, alternatively, at
<http://www.OpenLDAP.org/license.html>.

# ACKNOWLEDGEMENT
This work was initially developed by Simon Levermann <simon@slevermann.de>