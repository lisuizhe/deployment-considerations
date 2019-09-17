---
layout: default
css_id: key_management
---

# Key management

(Intro text here)

## Repository keys

On both the Director and the Image repository, the OEM maintains the keys to the Root, Timestamp, Snapshot, and Targets roles. However, for any delegated Targets roles on the Image repository, the corresponding keys are expected to be maintained by the delegated supplier. For example, If a tier-1 supplier signs its own images, then the supplier maintains its offline keys. Otherwise, the OEM would sign on behalf of the supplier, and thus maintain its keys.


### Online vs. offline keys

Repository administrators SHOULD use offline keys to sign the Root metadata on the Director repository, so attackers cannot tamper with this file after a repository compromise. The Timestamp, Snapshot, and Targets metadata SHOULD be signed using online keys, so that an automated process can instantly generate fresh metadata.

On the Image repository, the Timestamp and Snapshot metadata SHOULD be signed using online keys, so that an automated process can instantly generate fresh metadata. There are two options for signing the Timestamp and Snapshot metadata, each with the opposite trade-off from the other. In the first option, the OEM uses online keys meaning automated processes can automatically renew the Timestamp and Snapshot metadata to indicate that there are new Targets metadata and/or images available. However, attackers who have compromised a supplier's key would now have access to the Image repository and can instantly publish malicious images. If they additionally compromise the Director repository, then they can execute arbitrary software attacks by selecting these malicious images on the Image repository for installation. Such an attack could also facilitate mix-and-match attacks.

In the second option, the OEM uses offline keys to sign Timestamp and Snapshot metadata, which reduces the risk of attackers immediately publishing malicious images. The downside deals with expiration of Timestamp and Snapshot.  If the Timestamp and Snapshot metadata expire relatively quickly, then it is more cumbersome to use offline keys to renew their signatures. However, if a longer expiration time is used, then a man-in-the-middle attacker has more time with which to execute freeze attacks, hence defeating the purpose of the Timestamp role.

The keys to all other roles (Root, Targets, and all delegations) on the Image repository SHOULD be kept offline. This is because these metadata are expected to be updated relatively infrequently and so that a repository compromise does not immediately affect full verification ECUs. It does not matter where an offline key is stored (e.g., in a Hardware Security Module, YubiKey, or a USB stick in a safe deposit box), as long as the key is not accessible from the repository. Each key SHOULD be kept separately from others, so that a compromise of one does not affect them all.


### Key thresholds

#### Director repository

Since the Root role on the Director repository has the highest impact when its keys are compromised, it SHOULD use a sufficiently large threshold number of keys, so that a single key compromise is insufficient to sign its metadata file. Each key MAY belong to a repository administrator. For example, if there are 8 administrators, then at least 5 keys SHOULD be required to sign the Root metadata file, so that a quorum is required to trust the metadata.

The Timestamp, Snapshot, and Targets roles MAY each use a single key, because using more keys does not provide any additional security. If these keys are online, then attackers who compromise the repository can always use these online keys, regardless of their number.

##### Metadata expiration times

Since the Root role keys on the Director repository are not expected to be revoked and replaced often, its metadata file MAY expire after a relatively long time, such as one year.

The Timestamp, Snapshot, and Targets metadata files SHOULD expire relatively quickly, such as in a day, because they are used to indicate whether updated images are available.

Table 1 lists an example of expiration times for metadata files on the Director repository.

##### TODO: Insert Table 1 (expiration dates-director)

#### Image repository

For the Image repository, each role MAY use as many keys as is desired. However, the greater the impact when the keys for a role are compromised, then the greater the number of keys that it SHOULD use. Also, a threshold number of keys SHOULD be used, so that a single key compromise is generally insufficient to sign new metadata. To further increase compromise-resilience, each key SHOULD be unique across all roles.

Since the Root role has the highest impact when its keys are compromised, it SHOULD use a sufficiently large threshold number of keys. Each key MAY belong to a repository administrator. For example, if there are 8 administrators, then at least 5 keys SHOULD be required to sign the root metadata file, so that a quorum is required trust the metadata.

Since the Targets role also a high impact when its keys are compromised, it SHOULD also use a sufficiently large threshold number of keys. For example, 3 out of 4 keys MAY be required to sign the Targets metadata file.

Since the Timestamp and Snapshot roles have a relatively low impact when its keys are compromised, each role MAY use a small threshold number of keys. For example, each role MAY use 1 out of 2 keys to sign its metadata file.

Finally, each delegated Targets role SHOULD use at least 1 out of 2 keys to sign its metadata file, so that one key is available in case the other is lost. It is RECOMMENDED that the higher the number of ECUs that can be compromised when a delegated Targets role is compromised, then the higher the threshold number of keys that SHOULD be used to sign the role metadata.

##### Metadata expiration times

The Uptane Standards require all metadata files to have expiration times in order to prevent or limit freeze attacks. If ECUs know the time, then attackers can not indefinitely replay outdated metadata, and hence, images. In general, the expiration date for a metadata file depends on how often it is updated. The more often that it is updated, then the faster it SHOULD expire, so that attackers cannot execute freeze attacks. Even if it is not updated frequently, it SHOULD expire after a bounded period of time, so that stolen or lost keys can be revoked and replaced.

Since the Root role keys are expected to be revoked and replaced relatively rarely, its metadata file MAY expire after a relatively long time, such as one year.

Table 2 lists an example of expiration times for metadata files on the Image repository.

##### TODO: Insert Table 2 (expiration dates-image)


## What to do in case of key compromise

An OEM and its suppliers SHOULD be prepared to handle a key compromise. If the recommended number and type of keys are used, this should be a rare event. Nevertheless, when it happens OEMs and suppliers could use the following recovery procedures.

### Director repository

Since the Director repository MUST keep at least some software signing keys online, a compromise of this repository can lead to some security threats, such as mix-and-match attacks. Thus, the OEM SHOULD take great care to protect this repository, and reduce its attack surface as much as possible. For example, this MAY be done, in part, by using a firewall. However, if the repository has been compromised, then the following procedure SHOULD be performed in order to recover ECUs from the compromise.

First, the OEM SHOULD use the Root role to revoke and replace the keys to the Timestamp, Snapshot, and Targets roles, because only the Root role can replace these keys. Following the type and placement of keys prescribed for the Director repository, we assume that attackers have compromised the online keys to the Timestamp, Snapshot, and Targets roles, but not the offline keys to the Root role.

Second, the OEM SHOULD consider a manual recall of all vehicles in order to replace these keys, particularly if the vehicle has partial verification secondaries. This recall MAY be done by requiring vehicle owners to visit the nearest dealership. Although an OEM could replace these keys on a full verification ECU by using over-the-air broadcasts, a manual recall is recommended because:
1. the OEM SHOULD perform a safety inspection of the vehicles, in case of security attacks, and
2. partial verification secondaries are not designed to handle key revocation and replacement over-the-air. In order to update keys for partial verification secondaries, the OEM SHOULD overwrite their copies of the Root metadata file, perhaps using new images.

After inspecting the vehicle, the OEM SHOULD replace and update metadata and images on all ECUs to ensure that the images are known to be safe and that partial verification secondaries have replaced the keys for the Director repository.

### Image repository

If the recommendations for the type and placement of keys described above for the Image repository are followed, then a key compromise of this repository should be an unlikely event. However, should one occur, it is a much more serious affair. A compromise of the Image repository would allow attackers to tamper with images without being detected, and thus execute arbitrary software attacks. There are two cases for handling a key compromise, depending on whether the key is managed by a delegated supplier or by the OEM.

#### Supplier-managed keys

In the first case, where a tier-1 supplier or one of its delegatees has had one or more of its keys compromised, the supplier and its affected delegatees (if any), SHOULD revoke and replace keys. They SHOULD update metadata, including delegations and images, and send them to the OEM.

The OEM SHOULD then manually recall only affected vehicles that run software maintained by this supplier in order to replace metadata and images. This MAY be done by requiring vehicle owners to visit the nearest dealership. A manual recall SHOULD be done, because without trusted hardware (such as TPM), it is difficult to ensure that compromised ECUs can be remotely and securely updated. After having inspected the vehicle, the OEM SHOULD replace and update metadata and images on all ECUs so that these images are known to be safe.

#### OEM-managed keys

The second case, where the OEM has had a key compromised, is potentially far more serious than the first one. An attacker in such a position may be able to execute attacks on all vehicles depending on which keys have been compromised. If the keys are for the Timestamp and Snapshot roles, or the Targets role, or the Root role, then the OEM SHOULD use the following recovery procedure.

First, the OEM SHOULD use the Root role to revoke and replace keys for all affected roles. Second, it SHOULD restore all metadata and images on the Image repository to a known good state using an offline backup. Third, the OEM SHOULD manually recall all vehicles in order to replace metadata and images. A manual recall SHOULD be done, because without trusted hardware (such as TPM), it is difficult to ensure that compromised ECUs can be remotely and securely updated.

### ECU keys

If ECU keys are compromised, then the OEM SHOULD manually recall vehicles to replace these keys. This is the safest course of action because after a key compromise, an OEM cannot be sure whether it is remotely replacing keys controlled by attackers or the intended ECUs.

An OEM MAY use the Director repository and its inventory database to infer whether ECU keys have been compromised. Since the inventory database is used to record vehicle version manifests, which list what ECUs on a vehicle have installed over time, the OEM MAY use this information to detect abnormal patterns of installation that may have been caused by an ECU key compromise. Note, however, that this method is not perfect, because if attackers control ECU keys, then they can also use these keys to send fraudulent ECU version manifests.