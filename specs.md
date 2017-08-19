
## what

- SSH uploader to S3 servers
- hen the upload will get registered in Tierion
- runs in secure location (not close to IYO servers)
- easy python script (js9)

## what does it do
- backup the uploaded blob to 
   - X nr of S3 backends 
- now process the file 
   - put in at least 2 x S3 servers
   - register the action in Tierion
- info which is posted
   - metadata: any json file
   - the binary info
   - the name for which profile the registration happens
   
## the upload dir

- /upload/$profilename/$id_of_upload/
    - metadata.json
    - any nr of files, any format
- /profile/
    - $profilename.toml

## the upload ssh server

- is a docker with SSH server which can only be used for uploading
- configured in SSH server X nr of ssh pub keys which can be used to upload
- /upload/ is to upload any info 
- /profile/ is to upload profile's
- once processed then the /upload & /profile is empty again, in other words whoever uses the upload server can only upload the instructions, not retrieve what was registered

## process the dir

- tar.gz the dir to file
- hash file -> hash A (blake)
- hash file -> hash A' (other hash function, NOT BLAKE)
- encrypt file with hash A
- hash encrypted file: -> hash B
   - location in S3 $profilename/$month/$hashB
- register the action with Tierion
   - info submitted to Tierion 
       - $profilename/$id_of_upload
       - metadata (the json file)
       - hash A'
       - time epoch
       - size of file (after compression)
       - CRC of binary info (after compression)
       - signature of hash A' by private key of registrar 
       - description of registrar
       - [encrypted: hash A, ...] for each pub key known in the profile
       - [encrypted: hash B, ...]
       - previous record on tierion (for same $profilename/id_of_upload)
- encryption is done with: public key of registration profile (strongest possible)
- add the transaction to a local tlog file (tar format, one per day)
   - this means even if Tierion is not there we can out of the tar's reconstruct all info (is backup)
   - also means if the Registration server is compromised we are screwed !
   - location of tlog /$tlogpath/$profilename/$day.tar

## profile.toml

- pub key(s) of the persons who can retrieve the info
- description registrar (an be anything just to help anyone to find it back(
- S3 locations (multiple, with secrets)
- emails (will send report to the persons who can retrieve)

