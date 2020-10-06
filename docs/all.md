# All Auditor (all)

Runs all available auditors, or those specified using a kubeaudit config.

## General Usage

```
kubeaudit all [flags]
```

## Flags

| Short   | Long       | Description                               | Default                                  |
| :------ | :--------- | :---------------------------------------- | :--------------------------------------- |
| -k      | --kconfig  | Path to kubeaudit config                  |                                          |

Also see [Global Flags](/README.md#global-flags)

### Kubeaudit Config

The kubeaudit config can be used for two things:
1. Enabling only some auditors
1. Specifying configuration for auditors

Any configuration that can be specified using flags for the individual auditors can be represented using the config.

The config has the following format:

```yaml
enabledAuditors:
    # Auditors are enabled by default if they are not explicitly set to "false"
    apparmor: false
    asat: false
    capabilities: true
    hostns: true
    image: true
    limits: true
    mountds: true
    netpols: true
    nonroot: true
    privesc: true
    privileged: true
    rootfs: true
    seccomp: true
auditors:
    capabilities:
        # If no capabilities are specified and the 'capabilities' auditor is enabled,
        # drop ALL is set
        drop: 
        - ALL
    image:
        # If no image is specified and the 'image' auditor is enabled, WARN results
        # will be generated for containers which use an image without a tag
        image: "myimage:mytag"
    limits:
        # If no limits are specified and the 'limits' auditor is enabled, WARN results
        # will be generated for containers which have no cpu or memory limits specified
        cpu: "750m"
        memory: "500m"
```

For more details about each auditor, including a description of the auditor-specific configuration in the config, see the [Auditor Docs](/README.md#auditors).

**Note**: The kubeaudit config is not the same as the kubeconfig file specified with the `-c/--kubeconfig` flag, which refers to the Kubernetes config file (see [Local Mode](/README.md#local-mode)). Also note that only the `all` command supports using a kubeaudit config. It will not work with other commands.

## Examples

```
$ kubeaudit all -f "internal/test/fixtures/all_resources/deployment-apps-v1.yml"

---------------- Results for ---------------

  apiVersion: v1
  kind: Namespace
  metadata:
    name: deployment-apps-v1

--------------------------------------------

-- [error] MissingDefaultDenyIngressAndEgressNetworkPolicy
   Message: Namespace is missing a default deny ingress and egress NetworkPolicy.
   Metadata:
      Namespace: deployment-apps-v1


---------------- Results for ---------------

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: deployment
    namespace: deployment-apps-v1

--------------------------------------------

-- [error] AppArmorAnnotationMissing
   Message: AppArmor annotation missing. The annotation 'container.apparmor.security.beta.kubernetes.io/container' should be added.
   Metadata:
      Container: container
      MissingAnnotation: container.apparmor.security.beta.kubernetes.io/container

-- [error] AutomountServiceAccountTokenTrueAndDefaultSA
   Message: Default service account with token mounted. automountServiceAccountToken should be set to 'false' or a non-default service account should be used.

-- [error] CapabilityShouldDropAll
   Message: Capability not set to ALL. Ideally, you should drop ALL capabilities and add the specific ones you need to the add list.
   Metadata:
      Container: container
      Capability: AUDIT_WRITE

-- [error] NamespaceHostNetworkTrue
   Message: hostNetwork is set to 'true' in PodSpec. It should be set to 'false'.
   Metadata:
      PodHost:

-- [error] NamespaceHostIPCTrue
   Message: hostIPC is set to 'true' in PodSpec. It should be set to 'false'.
   Metadata:
      PodHost:

-- [error] NamespaceHostPIDTrue
   Message: hostPID is set to 'true' in PodSpec. It should be set to 'false'.
   Metadata:
      PodHost:

-- [warning] ImageTagMissing
   Message: Image tag is missing.
   Metadata:
      Container: container

-- [warning] LimitsNotSet
   Message: Resource limits not set.
   Metadata:
      Container: container

-- [error] RunAsNonRootPSCNilCSCNil
   Message: runAsNonRoot is not set in container SecurityContext nor the PodSecurityContext. It should be set to 'true' in at least one of the two.
   Metadata:
      Container: container

-- [error] AllowPrivilegeEscalationNil
   Message: allowPrivilegeEscalation not set which allows privilege escalation. It should be set to 'false'.
   Metadata:
      Container: container

-- [warning] PrivilegedNil
   Message: privileged is not set in container SecurityContext. Privileged defaults to 'false' but it should be explicitly set to 'false'.
   Metadata:
      Container: container

-- [error] ReadOnlyRootFilesystemNil
   Message: readOnlyRootFilesystem is not set in container SecurityContext. It should be set to 'true'.
   Metadata:
      Container: container

-- [error] SeccompAnnotationMissing
   Message: Seccomp annotation is missing. The annotation seccomp.security.alpha.kubernetes.io/pod: runtime/default should be added.
   Metadata:
      MissingAnnotation: seccomp.security.alpha.kubernetes.io/pod
```

### Example with Kubeaudit Config

Consider the following kubeaudit config `config.yaml`
```yaml
enabledAuditors:
    # Auditors are enabled by default if they are not explicitly set to "false"
    hostns: false
    image: false
auditors:
    capabilities:
    drop:
      - ALL
    add: 
    - AUDIT_WRITE
    - CHOWN
```

The config can be passed to the `all` command using the `-k/--kconfig` flag:
```
$ kubeaudit all -k "config.yaml" -f "auditors/all/fixtures/audit_all_v1.yml"
```

### Example with Flags

The behaviour of the `all` command can also be customized by using flags. The `all` command supports all flags supported by invididual auditors (see the individual [auditor docs](/README.md#auditors) for all the flags). For example, the `caps` auditor supports specifying capabilities to add with the `--add` flag so this flag can be used with the `all` command

```
kubeaudit all -f "auditors/all/fixtures/audit_all_v1.yml" --add "AUDIT_WRITE"
```
