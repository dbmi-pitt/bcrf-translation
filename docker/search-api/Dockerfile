# Parent image
FROM hubmap/api-base-image:latest

LABEL description="HuBMAP Search API Service"

# The commons branch to be used in requirements.txt during image build
# Default is master branch specified in docker-compose.yml if not set before the build
ARG COMMONS_BRANCH

# Change to directory that contains the Dockerfile
WORKDIR /usr/src/app

# Copy from host to image
COPY . .

# Nginx package from EPEL is old, we create a new repository file to install the latest mainline version of Nginx
RUN echo $'[nginx-mainline]\n\
name=nginx mainline repo\n\
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/\n\
gpgcheck=0\n\
enabled=1\n'\
>> /etc/yum.repos.d/nginx.repo

# Reduce the number of layers in image by minimizing the number of separate RUN commands
# 1 - Install nginx (using the custom yum repo specified earlier)
# 2 - Remove the default nginx config file
# 3 - Overwrite the nginx.conf with ours to run nginx as non-root
# 4 - Remove the copied nginx directory
# 5 - Install flask app dependencies with pip (pip3 also works)
# 6 - Make the start script executable
# 7 - Clean all yum cache
RUN yum install -y nginx && \
    rm /etc/nginx/conf.d/default.conf && \
    mv nginx/nginx.conf /etc/nginx/nginx.conf && \
    rm -rf nginx && \
    pip install -r src/search-api/src/requirements.txt && \
    chmod +x start.sh && \
    yum clean all 

# The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. 
# EXPOSE does not make the ports of the container accessible to the host.
# Here 5000 is for the uwsgi socket, 8080 for nginx
EXPOSE 5000 8080

# Set an entrypoint
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

CMD ["./start.sh"]
