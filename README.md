docker-munki
-----
A container that serves static files at http://munki/repo using nginx.

nginx expects the munki repo content to be located at /munki_repo. Use a data container and the --volumes-from option to add files.

Creating a Data Container:
---
Create a data-only container to host the Munki repo:  
	`docker run -d --name munki-data --entrypoint /bin/echo nmcspadden/munki Data-only container for munki`

For more info on data containers read [Tom Offermann](http://www.offermann.us/2013/12/tiny-docker-pieces-loosely-joined.html)'s blog post and the [official documentation](https://docs.docker.com/userguide/dockervolumes/). 

Run the Munki container:
-----
If you have an existing Munki repo on the host, you can mount that folder directly by using this option instead of --volumes-from:

`-v /path/to/munki/repo:/munki_repo`

Otherwise, use --volumes-from the data container:  
	`docker run -d --name munki --volumes-from munki-data -p 80:80 -h munki nmcspadden/munki`
	

Populate the Munki server (optional):
-----
The easiest way to populate the Munki server is to hook the munki-data volume up to a Samba container and share it out to a Mac, using my [SMB-Munki container](https://registry.hub.docker.com/u/nmcspadden/smb-munki/):  

1.	`docker pull nmcspadden/smb-munki`
2.	`docker run -dt -p 445:445 --volumes-from munki-data --name smb nmcspadden/smb-munki`
3.	You may need to change permissions on the mounted share, or change the samba.conf to allow for guest read/write permissions. One example:  
	`chown -R nobody:nogroup /munki_repo`  
	`chmod -R ugo+rwx /munki_repo`
4.	Populate the Munki repo using the usual tools - munkiimport, manifestutil, makecatalogs, etc.