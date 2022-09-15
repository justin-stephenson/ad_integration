# Direct AD Integration role template (https://github.com/linux-system-roles/template/)

An ansible role which configures direct Active Directory integration.

## Requirements

It is recommended to use the Administrator user to join with Active Directory. If the Administrator user cannot be used, the normal Active Directory user must have sufficient join permissions.

Time must be in sync with Active Directory servers.  The ad_integration role will use the timesync system role for this if the user specifies a parameter that allows the ad_integration role to manage timesync.  Parameters TBD.

RHEL8+ and Fedora no longer support RC4 encryption, it is recommended to enable AES in Active Directory, if not possible then the AD-SUPPORT crypto policy must be enabled.  The ad_integratoin role will use the crypto_policies system role for this if the user specifies a parameter that allows the ad_integration role to manage crypto_policies.

The Linux system must be able to resolve default AD DNS SRV records.

Ports must be opened from Linux to AD:

| Source Port | Destination | Protocol    | Service                                                   |
|-------------|-------------|-------------|-----------------------------------------------------------|
| 1024:65535  | 53          | TCP and UDP | DNS                                                       |
| 1024:65535  | 389         | TCP and UDP | LDAP                                                      |
| 1024:65535  | 636         | TCP         | LDAPS                                                     |
| 1024:65535  | 88          | TCP and UDP | Kerberos                                                  |
| 1024:65535  | 464         | TCP and UDP | Kerberos change/set password (kadmin)                     |
| 1024:65535  | 3268        | TCP         | LDAP Global Catalog (If "id_provider = ad" is being used) |
| 1024:65535  | 3269        | TCP         | LDAP Global Catalog SSL                                   |
| 1024:65535  | 123         | UDP         | NTP (Optional)                                            |


## Role Variables

**Required variables**
ad_integration_realm: realm/domain name to joinm

ad_integration_password: The password of the user used to authenticate with when joining the machine to the realm.  Do not use cleartext - use Ansible Vault to encrypt the value.

**Optional variables**
ad_integration_user: The user name to be used to authenticate with when joining the machine to the realm.
Default: Administrator

ad_integration_join_to_dc:  an Active Directory domain controller's hostname or IP address may be specified to join via that domain controller directly. If `ad_integration_manage_timesync` is set to true then this variable is required, time will also then be synchronized with this AD DC.
Default: Not set

ad_integration_auto_id_mapping: perform automatic UID/GID mapping for users and groups, set to `false` to rely on POSIX attributes already present in Active Directory.
Default: true

ad_integration_client_software: Only join realms for which we can use the given client software. Possible values include sssd or winbind. Not all values are supported for all realms.
Default: Automatic selection

ad_integration_membership_software: The software to use when joining to the realm. Possible values include samba or adcli. Not all values are supported for all realms.
Default: Automatic selection

ad_integration_computer_ou: The distinguished name of an organizational unit to create the computer account. It can be relative to the Root DSE, or a complete LDAP DN.
Default: Default AD computer container

ad_integration_manage_timesync: boolean - default false - If true, the ad_integration role will use fedora.linux_system_roles.timesync. Requires providing a value for `ad_integration_join_to_dc` to use as a time source.
 
ad_integration_manage_crypto_policies: boolean - default false - If true, the ad_integration role will use fedora.linux_system_roles.crypto_policies as needed

ad_integration_allow_rc4_crypto: boolean - default false - If true, the ad_integration role will set the crypto policy allowing RC4 encryption. Requires ad_integration_manage_crypto_policies to be set to true

## Dependencies

N/A

## Example Playbook

The following is an example playbook to setup direct Active Directory integration with AD domain “domain.example.com”, the join will be performed with user Administrator using the vault stored password. Prior to the join, the crypto policy for AD SUPPORT with RC4 encryption allowed will be set.

```yaml
- hosts: all
  vars:
    ad_integration_realm: "domain.example.com"
    ad_integration_password: !vault | …vault encrypted password…
    ad_integration_manage_crypto_policies: false
    ad_integration_allow_rc4_crypto: true
  roles:
    - linux-system-roles.ad_integration
```

## License

MIT.

## Author Information

Justin Stephenson (jstephen@redhat.com)