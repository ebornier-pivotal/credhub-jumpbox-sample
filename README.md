
This process was tested against PCF 1.11 alpha with only OpsManager + Bosh Director deployed.



**1. Install Credhub CLI**

```bash
wget https://github.com/cloudfoundry-incubator/credhub-cli/releases/download/0.8.0/credhub-linux-0.8.0.tgz
tar xvf credhub-linux-0.8.0.tgz
```

**2. Connect on Bosh Director UAA to create your own credhub user**

```
uaac target https://10.0.16.10:8443 --skip-ssl-validation
##get your credentials from Director Tiles / Credentials
## (client : login & user : admin)
uaac token owner get

uaac user add credhubuser
uaac group add credhub.write
uaac group add credhub.read
uaac member add credhub.read credhubuser
uaac member add credhub.write credhubuser

./credhub login -u credhubuser -p password
./credhub find

```

**3.  Prepare bosh environment (stemcell, jumpbox release)**

```
bosh --ca-cert /var/tempest/workspaces/default/root_ca_certificate target  10.0.16.10
# get your director credentials from Ops Manager
bosh login
bosh upload stemcell /var/tempest/stemcells/light-bosh-stemcell-3406-aws-xen-hvm-ubuntu-trusty-go_agent.tgz
bosh upload release https://bosh.io/d/github.com/cloudfoundry-community/jumpbox-boshrelease?v=4.2.17
```

**4. Have a look to the templated manifest**

```
cat jumpbox.yml
```

**5. Set variables thru credhub**

```
./credhub set -n  /p-bosh/jumpbox/hostname -t value -v myhostname
./credhub generate -n /p-bosh/jumpbox/jumpbox-key -t ssh
bosh deploy
```

**6. SSH to jumpbox vm customized**

```bash
bosh ssh
Last login: Mon May 29 13:39:00 2017 from 10.0.0.248
[13:39:00] bosh_jby3j52vr@myhostname ~

```
