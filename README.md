# arin-roa-request
A scripts to connect to ARIN's RESTful API to create ROAs

Note! The script ote-arin-roa-request.py is the same as the arin-roa-request.py script except it makes the API call to ARIN's OT&E (Operational Test & Evaluation) Environment.  Any change made in the OT&E is just a test and does not impact the "real" environment.

In order to execute the script you will need to generate an API key.  See this link to find out more info on generating an API key with ARIN: https://www.arin.net/reference/materials/security/api_keys/

You will also need a private key that will be used to sign your ROAs.  The OT&E has a special private key that is used for signing all ROAs in that test environment.  The test private key can be downloaded here: https://www.arin.net/reference/tools/testing/ote_roa_req_signing_key.private.pem More info is here: https://www.arin.net/reference/tools/testing/#roa-request-generation-key-pairs

For production, you will need to generate your own public/private key pair.  This procedure is documented at https://www.arin.net/resources/manage/rpki/roa_request/
Here’s the procedure to quickly create a ROA in ARIN using your browser:
Create your public/private key pair using OpenSSL:
```openssl genrsa -out org keypair.pem 2048```
This command generates a ROA Request Generation Key Pair and saves it as a file named orgkeypair.pem.
```openssl rsa -in orgkeypair.pem -pubout -outform PEM -out org_pubkey.pem```
This command extracts the public key from the ROA Request Generation key pair and writes it to a file named org_pubkey.pem.
Keep the orgkeypair.pem file private, perhaps in an HSM.  If the security of the private key is compromised, you should delete all of the ROAs created with that key and generate a new key pair and new ROAs.

The script needs to have a CSV file specified which defines the values for each ROA that is created.  The format of the CSV is:
Origin AS,IP prefix,CIDR mask,maxLength
Ex: 65000,192.0.2.0,24,24
Note, the maxLength is required in this script!  It is recommended that the maxLength be equal to the CIDR mask.  If you set the maxLength too large (ex. /24 or /48) you open yourself up to a potential forged-origin subprefix hijack (see this IETF doc: https://tools.ietf.org/html/draft-ietf-sidrops-rpkimaxlen)

The script can be run like this:
```./ote-arin-roa-request.py -c ROAs.txt -a <ARIN API KEY> -k ote_roa_req_signing_key.private.pem -o <ORG-ID> --debug
