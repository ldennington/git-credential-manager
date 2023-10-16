# Validate Linux GPG signature

## Debian package

```shell
# Install needed packages
apt-get install -y curl debsig-verify

# Download signature file
curl -s https://api.github.com/repos/git-ecosystem/git-credential-manager/releases/latest \
| grep "browser_download_url.*asc" \
| cut -d : -f 2,3 \
| tr -d \" \
| xargs -I 'url' curl -L -o gcm.asc 'url'

# De-armor signature file
gpg --output gcm.gpg --dearmor gcm.asc

# Note that the fingerprint of this key is "3C853823978B07FA", which you can
# determine by running:
gpg --show-keys gcm.asc | head -n 2 | tail -n 1 | tail -c 17

# Copy de-armored public key to debsig keyring folder
mkdir /usr/share/debsig/keyrings/3C853823978B07FA
mv gcm.gpg /usr/share/debsig/keyrings/3C853823978B07FA/

# Create an appropriate policy file
mkdir /etc/debsig/policies/3C853823978B07FA
cat > /etc/debsig/policies/3C853823978B07FA/generic.pol << EOL
<?xml version="1.0"?>
<!DOCTYPE Policy SYSTEM "https://www.debian.org/debsig/1.0/policy.dtd">
<Policy xmlns="https://www.debian.org/debsig/1.0/">

  <Origin Name="Git Credential Manager" id="3C853823978B07FA" Description="Git Credential Manager public key"/>

  <Selection>
    <Required Type="origin" File="gcm.gpg" id="3C853823978B07FA"/>
  </Selection>

  <Verification MinOptional="0">
    <Required Type="origin" File="gcm.gpg" id="3C853823978B07FA"/>
  </Verification>

</Policy>
EOL

# Now download the Debian package and verify it.
curl -s https://api.github.com/repos/git-ecosystem/git-credential-manager/releases/latest \
| grep "browser_download_url.*deb" \
| cut -d : -f 2,3 \
| tr -d \" \
| xargs -I 'url' curl -L -o gcm.deb 'url'
debsig-verify gcm.deb
```