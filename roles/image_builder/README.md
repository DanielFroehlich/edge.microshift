Role Name
=========

This role automates the setup server and [osbuild](https://www.osbuild.org/) [compose builds](https://www.osbuild.org/guides/user-guide/user-guide.html) using the osbuild backend [Weldr](https://weldr.io/) API with Microshift and necessary dependencies installed.

It uses the [infra.osbuild.builder](https://github.com/redhat-cop/infra.osbuild/blob/main/roles/builder/README.md) role, see also the docs over there.


Requirements
------------

None

Role Variables
--------------

## microshift_rhsm_repos

Type: list
Required: false

*Use microshift_rhsm_repos to set the rhsm repos used for the Microshift package*

List of [RHSM](https://access.redhat.com/products/red-hat-subscription-management/) repositories to make available to the
[osbuild](https://www.osbuild.org/) [compose builds](https://www.osbuild.org/guides/user-guide/user-guide.html).

Example:

```yaml
microshift_rhsm_repos:
    - "rhocp-4.12-for-rhel-{{ ansible_distribution_major_version }}-{{ ansible_architecture }}-rpms"
    - "fast-datapath-for-rhel-{{ ansible_distribution_major_version }}-{{ ansible_architecture }}-rpms"
```

## microshift_image_custom_repos

Type: list
Required: false

*Use microshift_image_custom_repos to use community repos for the Microshift package*

List of custom repositories to make available to the
[osbuild](https://www.osbuild.org/) [compose builds](https://www.osbuild.org/guides/user-guide/user-guide.html).

Example:

```yaml
microshift_image_custom_repos:
  - name: EPEL8
    base_url: "https://dl.fedoraproject.org/pub/epel/{{ hostvars[inventory_hostname].ansible_distribution_major_version }}/Everything/x86_64/"
    type: yum-baseurl
    check_ssl: true
    check_gpg: false
  - name: microshift
    base_url: https://download.copr.fedorainfracloud.org/results/@redhat-et/microshift-testing/epel-8-x86_64/
    type: yum-baseurl
    check_ssl: true
    check_gpg: false
  - name: microshift-deps
    base_url: https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rpms/4.12-el8-beta/
    type: yum-baseurl
    check_ssl: true
    check_gpg: false
```

### microshift_image_blueprint_name

Type: string
Required: false

This is the name of the [osbuild blueprint](https://www.osbuild.org/guides/blueprint-reference/blueprint-reference.html?highlight=distro#distribution-selection-with-blueprints)
to use. The blueprint will be auto generated based on the contents of the
`microshift_image_compose_customizations` role variable. In the event an of an [rpm-ostree](https://rpm-ostree.readthedocs.io/en/stable/)
based compose type specified by the `microshift_image_compose_type` role variable, the
blueprint name defined in this variable will use used to define the resulting [ostree](https://ostreedev.github.io/ostree/)
repository.

### microshift_image_blueprint_src_path

Type: string
Required: false

This is the path to a location on the osbuild server that the generated
blueprint should be stored at and used as the source content for the osbuild
compose build.

### microshift_image_compose_type

Type: string
Required: false

This variable defines the type of compose desired, valid inputs will vary based
on operating system (RHEL, CentOS Stream, or Fedora) and release version therin.

For RHEL, the Red Hat Enterprise Linux Documentation Team publishes these and they can be found [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/composing_a_customized_rhel_system_image/index#composer-output-formats_composer-description).

For CentOS Stream and Fedora, you will need to reference the output of the
`composer-cli compose types` command on the osbuild server (this can also be
done on RHEL if preferred).

### microshift_image_pubkey

Type: string
Required: false

Ssh public key to inject into the resulting image to allow
key-based ssh functionality without extra configuration for systems installed
with the resulting build media. If needed, use lookup to to get the contents from the public ssh key file.

Example:
```yaml
microshift_image_pubkey: "{{ lookup('file', '~/.ssh/id_rsa.pub', errors='warn') }}"
```

### microshift_image_compose_customizations:

Type: dict
Required: false

This variable is the YAML dict expression of
[osbuild blueprint](https://www.osbuild.org/guides/blueprint-reference/blueprint-reference.html) customizations.

Example:
```yaml
microshift_image_compose_customizations:
  user:
    name: "testuser"
    description: "test user"
    password: "testpassword"
    key: "{{ microshift_image_pubkey_file }}"
    groups: '["users", "wheel"]'
  kernel:
    append: "nomst=force"
  services:
    enabled: ["firewalld"]
  firewalld.services:
    enabled: ["ssh", "https"]

```

### microshift_image_firewall_options

Type: complex
Required: false

Custom list of firewall options

Each list entry is a [YAML dictionary](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
type and has the following attributes:

| Variable Name | Type                              | Required  | Default Value |
|---------------|-----------------------------------|-----------|---------------|
| zone          | string                            | **Yes**   | n/a           |
| source        | string                            | **Yes**   | n/a           |
| port          | string                            | **Yes**   | n/a           |

NOTES:

Only either source or port can be used at a time.

Example:

```yaml
microshift_image_firewall_options:
  - zone: trusted
    source: 169.254.169.1
  - zone: trusted
    port: 443/tcp
```

### microshift_image_lvms_pvc_name

Type: string
Required: false

Name used for PVC metadata

Example:

```yaml
microshift_image_lvms_pvc_name: my-lv-pvc
```

### microshift_image_lvms_pvc_access_modes

Type: string
Required: false

Access mode of the PVC

Example:
```yaml
microshift_image_lvms_pvc_access_modes: ReadWriteOnce
```

### microshift_image_lvms_pvc_storage

Type: string
Required: false

PVC storage size

Example:
```yaml
microshift_image_lvms_pvc_storage: 1G
```

### microshift_image_lvms_pod_name

Type: string
Required: false

Name used for pod metadata

Example:
```yaml
microshift_image_lvms_pod_name: my-pod
```

### microshift_image_lvms_pod_containers

Type: dict
Required: false

Containers spec

Example:
```yaml
microshift_image_lvms_pod_containers:
    name: nginx
    image: nginx
    command: '["/usr/bin/sh". "-c"]'
    args: '["sleep", "1h"]'
    volumeMounts:
        mountPath: /mnt
        name: my-volume
```

### microshift_image_lvms_pod_volumes

Type: string
Required: false

Volumes spec

Example:
```yaml
microshift_image_lvms_pod_volumes:
    name: my-volume
    claimName: my-lv-pvc
```

### microshift_image_gateway_interface

Type: string
Required: false

Ingress that is the API gateway

Example:
```yaml
microshift_image_gateway_interface: eth0
```

### microshift_image_external_gateway_interface

Type: string
Required: false

Ingress routing external traffic to your services and pods inside the node

Example:
```yaml
microshift_image_external_gateway_interface: eth1
```

### microshift_image_mtu

Type: string
Required: false



MTU value used for the pods

Example:
```yaml
microshift_image_mtu: 1400
```

### microshift_image_crio_proxy

Type: complex
Required: false

Options for deploying a microshift cluster behind an http(s) proxy

microshift_image_crio_proxy is a [YAML dictionary](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
type and has the following attributes:

| Variable Name | Type                              | Required  | Default Value |
|---------------|-----------------------------------|-----------|---------------|
| user          | string                            | **Yes**   | n/a           |
| password      | string                            | **Yes**   | n/a           |
| server        | string                            | **Yes**   | n/a           |
| port          | string                            | **Yes**   | n/a           |

NOTES:

Using this option will inject credentials into kickstart that is exposed via http

Example:

```yaml
microshift_image_crio_proxy:
  user: user1
  password: pass1
  server: 192.183.3.333
  port: 123
```

### microshift_image_pull_secret

Type: file / string
Required: false

Pull secret allows authentication with the container registries that serve the container images used by the official Red Hat supported MicroShift.

For downloading the pull secret from the Red Hat Hybrid Cloud Console, click [here](https://console.redhat.com/openshift/install/pull-secret)

Example:
```yaml
microshift_image_pull_secret: "{{ lookup('file', '~/pull-secret') }}"
```

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    ---
    - name: Run microshift image builder role
      become: true
      hosts: all
      gather_facts: true
      tasks:
        - name: Create image with microshift
          ansible.builtin.import_role:
            name: edge.microshift.image_builder

License
-------

GPLv3
