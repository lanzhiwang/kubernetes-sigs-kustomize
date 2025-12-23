[Kustomize Components](https://kubectl.docs.kubernetes.io/guides/config_management/components/)

```bash
$ DEMO_HOME=~/work/code/go_code/kustomize/learn/kustomize_components

$ BASE=${DEMO_HOME}/base
$ mkdir ${BASE}
$ touch ${BASE}/kustomization.yaml
$ touch ${BASE}/deployment.yaml
$ tree -a .
.
├── README.md
└── base
    ├── deployment.yaml
    └── kustomization.yaml

2 directories, 3 files
$

$ EXT_DB=${DEMO_HOME}/components/external_db
$ mkdir -p ${EXT_DB}
$ touch ${EXT_DB}/kustomization.yaml
$ touch ${EXT_DB}/deployment.yaml
$ touch ${EXT_DB}/configmap.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
└── components
    └── external_db
        ├── configmap.yaml
        ├── deployment.yaml
        └── kustomization.yaml

4 directories, 6 files
$

$ LDAP=${DEMO_HOME}/components/ldap
$ mkdir -p ${LDAP}
$ touch ${LDAP}/kustomization.yaml
$ touch ${LDAP}/deployment.yaml
$ touch ${LDAP}/configmap.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
└── components
    ├── external_db
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── ldap
        ├── configmap.yaml
        ├── deployment.yaml
        └── kustomization.yaml

5 directories, 9 files
$

$ RECAPTCHA=${DEMO_HOME}/components/recaptcha
$ mkdir -p ${RECAPTCHA}
$ touch ${RECAPTCHA}/kustomization.yaml
$ touch ${RECAPTCHA}/deployment.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
└── components
    ├── external_db
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    ├── ldap
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── recaptcha
        ├── deployment.yaml
        └── kustomization.yaml

6 directories, 11 files
$

$ COMMUNITY=${DEMO_HOME}/overlays/community
$ mkdir -p ${COMMUNITY}
$ touch ${COMMUNITY}/kustomization.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
├── components
│   ├── external_db
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   ├── ldap
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   └── recaptcha
│       ├── deployment.yaml
│       └── kustomization.yaml
└── overlays
    └── community
        └── kustomization.yaml

8 directories, 12 files
$

$ ENTERPRISE=${DEMO_HOME}/overlays/enterprise
$ mkdir -p ${ENTERPRISE}
$ touch ${ENTERPRISE}/kustomization.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
├── components
│   ├── external_db
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   ├── ldap
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   └── recaptcha
│       ├── deployment.yaml
│       └── kustomization.yaml
└── overlays
    ├── community
    │   └── kustomization.yaml
    └── enterprise
        └── kustomization.yaml

9 directories, 13 files
$

$ DEV=${DEMO_HOME}/overlays/dev
$ mkdir -p ${DEV}
$ touch ${DEV}/kustomization.yaml
$ tree -a .
.
├── README.md
├── base
│   ├── deployment.yaml
│   └── kustomization.yaml
├── components
│   ├── external_db
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   ├── ldap
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   └── recaptcha
│       ├── deployment.yaml
│       └── kustomization.yaml
└── overlays
    ├── community
    │   └── kustomization.yaml
    ├── dev
    │   └── kustomization.yaml
    └── enterprise
        └── kustomization.yaml

10 directories, 14 files
$

```
