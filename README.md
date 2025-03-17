# businesscard.ops

This repositry regroups the configuration-as-code to provision, configure,
deploy and manage the EPFL's businesscard app. It uses [Ansible] wrapped in a
convenient [suitcase], called [`busible`](./busible).


## Prerequisites

* Be member of the [Keybase] `/keybase/team/epfl_bsncrd/` team.
* Be member of the EPFL group `vra_p_svc0077`.


## Deploy

```bash
./busible      # (--prod for production environment)
```

Start a new "cloud" build

```bash
./busible -t image.startbuild
```

Restart the app

```bash
./busible -t app.restart`    # (--prod for production environment)
```


[Ansible]: https://www.ansible.com (Ansible is Simple IT Automation)
[suitcase]: https://github.com/epfl-si/ansible.suitcase (Install Ansible and its dependency stack into a temporary directory)
[github]: https://github.com/epfl-si/businesscard
[Keybase]: https://keybase.io
