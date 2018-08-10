# Manually adding MAAS clouds

A MAAS cloud may be registered with Juju using the interactive `add-cloud`
command (see [Using MAAS with Juju][clouds-maas]) but it is also possible to do
so using a YAML file. All that varies is the endpoint and the name, since they
use the same authentication method. Here is an example:

```yaml
clouds:
   devmaas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://devmaas/MAAS
   testmaas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://172.18.42.10/MAAS
   prodmaas:
      type: maas
      auth-types: [oauth1]
      endpoint: http://prodmaas/MAAS
```

This example YAML defines three MAAS (region) controllers. To add a MAAS cloud
from this definition to Juju, run the command in the form:
 
```bash
juju add-cloud <cloudname> <YAML file>
```

To add two MAAS clouds from the above example we would run:

```bash
juju add-cloud devmaas maas-clouds.yaml
juju add-cloud prodmaas maas-clouds.yaml
```

Where the supplied cloud names refer to those in the YAML file.

This will add both the 'prodmaas' and 'devmaas' clouds, which you can confirm
by running:
 
```bash
juju list-clouds
```

This will list the newly added clouds:

<!-- JUJUVERSION: 2.0.0-xenial-amd64 -->
<!-- JUJUCOMMAND: juju list-clouds -->
```no-highlight
Cloud        Regions  Default        Type        Description
aws               11  us-east-1      ec2         Amazon Web Services
...
devmaas            0                 maas        Metal As A Service
prodmaas           0                 maas        Metal As A Service
testmaas           0                 maas        Metal As A Service
```

It is necessary to add credentials for these clouds before bootstrapping them.
See the [Documentation on MAAS credentials here][maas-credentials].


<!-- LINKS -->

[clouds-maas]: ./clouds-maas.md
[maas-credentials]: ./clouds-maas.md#add-credentials
