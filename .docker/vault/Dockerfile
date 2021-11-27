# Base image
FROM    alpine:3.7

# Set vault version
ENV     VAULT_VERSION 1.9.0

# Create a new directory
RUN     mkdir -p /usr/src/vault \
        && cd /usr/src/vault \
        && mkdir config data logs policies
        

# Download dependencies and set up vault
RUN     apk --no-cache add ca-certificates wget \
        && wget --quiet --output-document=/tmp/vault.zip https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip \
        && unzip /tmp/vault.zip -d /usr/src/vault/ \
        && rm -f /tmp/vault.zip \
        && chmod +x /usr/src/vault

# Update PATH
ENV     PATH="PATH=$PATH:/usr/src/vault/"

# Add the config file
ADD     config/vault-config.json /usr/src/vault/config/vault-config.json

# Expose port
EXPOSE  8200

# Run vault
ENTRYPOINT ["vault"]
