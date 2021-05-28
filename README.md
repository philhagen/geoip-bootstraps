# geoip-bootstraps

Recent changes to the licensing of MaxMind's GeoIP databases requires users to create a login even to download the free products, as well as to update them.  This is incredibly inconvenient, as it breaks any platform that a developer intends to deploy in an operational state.

In this repository are several methods to streamline the process of obtaining and updating the databases without circumventing the requirement for a MaxMind account and license key.

These approaches are intended to be customized and integrated to your own platform or application workflow.  These scripts are not intended to be used by themselves.  I'm pretty sure they won't help you at all in a standalone capacity.

## Prerequisites

1. The `geoipupdate` tool must be installed.
2. A MaxMind account is required.  [Sign up for a free account here](https://www.maxmind.com/en/geolite2/signup).
3. A MaxMind license key is required.  After signing up for and into your MaxMind account, [generate a license key here](https://www.maxmind.com/en/accounts/155942/license-key).  Note that you'll need to know the version of the `geoipupdate` utility you'll be using.

## Method 1: User-initiated Download

This method uses an artificially created file containing a valid GeoIP database shell but no content.  Most GeoIP-using tools will accept this at startup but you should test this on your own.  After starting with the empty GeoIP database files, users run a script which requests their MaxMind user id and license key, then creates a `GeoIP.conf` file and downloads the databases.  This requires the `geoipupdate` tool is installed and configured to use the `GeoIP.conf` file that will be generated.  (You will likely need to adjust paths in the script for it to work on your platform.)  Subsequent runs of the `geoipupdate` tool will retrieve new copies of the GeoIP database files using the same user id and license key.

1. Place the `geoip_bootstrap.sh` and `GeoIP.conf.default` files on the system.
2. Install the included [empty.mmdb](empty.mmdb) file wherever a GeoIP database is required.
3. Tailor the `geoip_bootstrap.sh` script's `geoip_conf_template` and `geoip_conf_target` variables.
4. Ensure the system does not have a `GeoIP.conf` file in the location specified by `geoip_conf_target`.
5. Direct users to run `geoip_bootstrap.sh` after booting the platform when GeoIP features are required.
6. Before distributing the platform, ensure any GeoIP databases are replaced with `empty.mmdb` and the `GeoIP.conf` file has been deleted.

## Method 2: Boot-initiated Download

This method uses `systemd` to run a script at boot time which requests the user's GeoIP user id and license key, creates a `GeoIP.conf` file, then retrieves the GeoIP databases before completing the boot process.

NOTICE: This method WILL hang the boot process for 30 seconds before continuing without creating a `GeoIP.conf` file or downloading the MaxMind databases.

1. Place the `geoip_bootstrap.sh` and `GeoIP.conf.default` files on the system.
2. Install the included [empty.mmdb](empty.mmdb) file wherever a GeoIP database is required. (Optional but recommended.)
3. Tailor the `geoip_bootstrap.sh` script's `geoip_conf_template` and `geoip_conf_target` variables.
4. Ensure the system does not have a `GeoIP.conf` file in the location specified by `geoip_conf_target`.
5. Place the `geoip_bootstrap.service` file on the system, usually in the `/etc/systemd/system/` directory.
6. Tailor the `geoip_bootstrap.service` file's `Before=` and `RequiredBy=` parameters as needed to to ensure proper service startup order.
7. Run `systemctl daemon-reload`.
8. Run `systemctl enable geoip_bootstrap.service`.
9. During the next boot cycle, the script will interrupt the boot process, setting up GeoIP.
10. Before distributing the platform, ensure any GeoIP databases are replaced with `empty.mmdb` and the `GeoIP.conf` file has been deleted.
