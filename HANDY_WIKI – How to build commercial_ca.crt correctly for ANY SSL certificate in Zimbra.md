Zimbra SSL – How to Create commercial_ca.crt (Universal Method)

Use this whenever you get error
“unable to get local issuer certificate”

1️⃣ Go to SSL directory
su - zimbra
cd /opt/zimbra/ssl


Place your files here:

commercial.key

crt.crt (your server certificate)

2️⃣ Verify Key & Certificate match
/opt/zimbra/bin/zmcertmgr verifycrt comm ./commercial.key ./crt.crt ./commercial_ca.crt


If CA chain is wrong, you will get error 20.

3️⃣ Find correct Issuer (Intermediate CA)
openssl x509 -in ./crt.crt -noout -issuer -subject


Example output:

issuer= C = BE, O = GlobalSign nv-sa, CN = GlobalSign GCC R6 AlphaSSL CA 2025

4️⃣ Extract CA download URL directly from certificate

(This avoids broken vendor links forever)

openssl x509 -in ./crt.crt -noout -text | grep -A2 "CA Issuers"


Example output:

CA Issuers - URI:http://secure.globalsign.com/cacert/gsgccr6alphasslca2025.crt


Download it:

wget http://secure.globalsign.com/cacert/gsgccr6alphasslca2025.crt

5️⃣ Convert Intermediate CA to PEM
openssl x509 -inform DER -in gsgccr6alphasslca2025.crt -out intermediate.pem

6️⃣ Extract ROOT CA URL from Intermediate
openssl x509 -in intermediate.pem -noout -text | grep -A3 "CA Issuers"


Example:

CA Issuers - URI:http://secure.globalsign.com/cacert/root-r6.crt


Download it:

wget http://secure.globalsign.com/cacert/root-r6.crt

7️⃣ Convert Root CA to PEM
openssl x509 -inform DER -in root-r6.crt -out root.pem

8️⃣ Build CA bundle (order is CRITICAL)
cat intermediate.pem root.pem > commercial_ca.crt

9️⃣ Verify chain
/opt/zimbra/bin/zmcertmgr verifycrt comm ./commercial.key ./crt.crt ./commercial_ca.crt


You must see:

Certificate chain verified successfully.

🔟 Deploy SSL in Zimbra
cp ./crt.crt /opt/zimbra/ssl/zimbra/commercial/commercial.crt
cp ./commercial.key /opt/zimbra/ssl/zimbra/commercial/commercial.key
cp ./commercial_ca.crt /opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt

/opt/zimbra/bin/zmcertmgr deploycrt comm ./crt.crt ./commercial_ca.crt
zmcontrol restart

✅ Final Test
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com


Bottom must show:

Verify return code: 0 (ok)

This method works for:

GlobalSign

DigiCert

Sectigo / Comodo

GeoTrust

GoDaddy

Let’s Encrypt

ANY commercial CA

Just follow the certificate’s own CA Issuers links – no guessing ever again.
