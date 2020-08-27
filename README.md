# badger-chainz

See https://simple-evcorr.github.io/

#.............................................!!!!!!................................................#

WARNING: some web server configurations will fail to restart the web server if the FIFO is in place.

If your configuration breaks after setup during a web server (httpd, apache2, nginx) restart, then you will either need to fix that (permssions? etc) or switch
it up and actually write a log file: edit the scripts to not rm the log and mkfifo it. Doing this work around is not acceptible for some cases when you don't want to write the access data to the disk ever, but in the case where that is permited, an alternative is to have a rule or function at the end of the rule chain that truncates the log file. This work around does have some risks for data loss if very close rapid valid requests come through, so throttling the truncate could be helpful potentially, and this is the same reason that the badger-chainz customer rule template has window=1 which limits to max triggers to 1 per second. If you want to scale out to go beyond 1/tps badger-chainz, I recommend ordered load balancing so that the chains can be reassembled/inserted by another service such as using haproxy leastconn algorithm "balance leastconn" in the haproxy config and each chain server in that pool. Then the scaled out collection service (not included here) reads off the chains themselves and not the private keys, and then encrypts them again if needed, and inserts them into, say distributed cloud databases. Your data scientists may get upset when they query and all of the actual data is encrypted but so will the hackers, and this is how we granularly control access to sensitive data that is replicated to many unknown disks in the cloud etc. This tool isn't really meant to scale much beyond that, and would be rewritten in rust etc to be actually good. This is more of a in-some-cases-functional prototype to template the implementation of a strict and complex data lifecycle. Anyone or any software that wants to actually use the data will have to have access to keys, and in some cases layers of shared and secret keys, to use the data at all.

#.............................................!!!!!!................................................#


I have got FIFO access.log working so far with Debian and NGINX. I have seen many default configurations for httpd and apache2 fail to restart the web service when the FIFO is in place.

replace access.log with a FIFO (named pipe of the same name), read the data off the FIFO, encrypt then signed transaction chain.

Important note about default badger-chainz and alternate configurations:

With badger-chainz, the data is always stored encrypted but the transaction keys on the server are not with the default badger-chainz. This means that this transaction chain component does not provide much secrecy in terms of server side data storage, but it does provide a layer of client validation, data control (for purging individual sources, managing sources independently), and method for encrypted data distribution: all of the data for a specific customer or device can be sent to them encrypted in a way that they can decrypt. To solve for this, we'll layer up GPG/PGP on top of this.

If you need to add data storage secrecy, edit then deploy the badger-chainz-secret-mode to /usr/local/sbin/badger-chainz
The edit is to ensure the email address or identifier for the gpg public key is set in the script where the template has "PGPEMAIL".
You will need to have a gpg/pgp public key imported into the user's keyring, and not have that private key anywhere on the server.
This will mean that the admin will have to pull the encrypted chain down to his decryption device that has the gpg private key to decrypt
the chain and do anything with it. Use this if top security is needed and when possible.

Not using secret mode does not make it insecure, but certainly the default is more focused on identity, granularity, and validation.

The third mode is not actually badger-chainz, but should be used if you don't need to create the transaction chain and you just need to encrypt the data.
This mode is badger-weblog and it has a separate installer install-badger-weblog. The badger-weblog setup doesn't separate or validate anything, doesn't create
the blockchain, but uses the FIFO for the access.log and just writes all of the log data encrypted to badger-weblog_access.log.$stamp.enc

The install scripts do not expect you to use badger-chainz blockchain AND badger-weblog, one or the other. You can modify the scripts to run both if you really want to, but the design is to use badger-chainz-secret-mode if you want both together.



