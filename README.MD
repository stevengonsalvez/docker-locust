# Docker file

### Why do we have a customised docker file - rather than use the standard locustio build ?

When you write your locust file with complex operations and import other packages , the locustio image will not work (as it only contains the standrad build)

To enable you to add in packages to be available for your `locustfile.py` to use we have this image build over here , just update the requirements.txt with your packages and execute the below


`docker build -t <repo>:<tag>`

> Note the image is built from a default locust file. 
