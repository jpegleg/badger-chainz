# badger-chainz

See https://simple-evcorr.github.io/


replace access.log with a FIFO (named pipe of the same name), read the data off the FIFO with SEC, encrypt then signed transaction chain.

Important note about default badger-chainz and alternate configurations:

With badger-chainz, the data is always stored encrypted but the transaction keys on the server are not with the default badger-chainz. This means that this transaction chain component does not provide much secrecy in terms of server side data storage, but it does provide a layer of client validation, data control (for purging individual sources, managing sources independently), and method for encrypted data distribution: all of the data for a specific customer or device can be sent to them encrypted in a way that they can decrypt. To solve for this, we'll layer up GPG/PGP on top of this.

If you need to add data storage secrecy, edit then deploy the badger-chainz-secret-mode to /usr/local/sbin/badger-chainz
The edit is to ensure the email address or identifier for the gpg public key is set in the script where the template has "PGPEMAIL".
You will need to have a gpg/pgp public key imported into the user's keyring, and not have that private key anywhere on the server.
This will mean that the admin will have to pull the encrypted chain down to his decryption device that has the gpg private key to decrypt
the chain and do anything with it. Use this if top security is needed and when possible. 

Not using secret mode does not make it insecure, but certainly the default is more focused on identity, granularity, validation, and usability.

The third mode is not actually badger-chainz, but should be used if you don't need to create the transaction chain and you just need to encrypt the data.
This mode is badger-weblog and it has a separate installer install-badger-weblog. The badger-weblog setup doesn't separate or validate anything, doesn't create
the blockchain, but uses the FIFO for the access.log and just writes all of the log data encrypted to badger-weblog_access.log.$stamp.enc

The install scripts do not expect you to use badger-chainz blockchain AND badger-weblog, one or the other. You can modify the scripts to run both if you really want to, but the design is to use badger-chainz-secret-mode if you want both together.

And if you just don't want to have logs stored and want to be able to leverage the FIFO

rm -f /var/log/nginx/access.log && mkfifo /var/log/nginx/access.log
rm -f /var/log/httpd/access.log && mkfifo /var/log/httpd/access.log
rm -f /var/log/apache2/access.log && mkfifo /var/log/apache2/access.log

You might just do that to create privacy for your users and not store any log files. Then if you need to debug, you can attach a SEC process or simply cat the FIFO.


WARNING: apache2 gets upset about the fifo in some situations and will fail to start! Removing the fifo during restart is one solution:

rm -f /var/log/apache2/access.log && systemctl restart apache2 && rm -f /var/log/apache2/access.log && mkfifo /var/log/apache2/access.log

