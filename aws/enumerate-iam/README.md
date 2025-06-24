# Adjustments to `enumerate-iam`

The intention of this article is to describe how an Operator can adjust the tool [enumerate-iam](https://github.com/andresriancho/enumerate-iam) to only target specific AWS services.

Amazon Web Services (AWS) contains, as of 2025, 240 fully featured services<sup><a href="https://www.aboutamazon.com/what-we-do/amazon-web-services" target="_blank" rel="noopener noreferrer">1</a></sup>. In the event AWS cloud credentials are capturing during a capture the flag or authorized engagement the permissions for the credentials will be unknown. This is where the tool can assist. 

[enumerate-iam](https://github.com/andresriancho/enumerate-iam) will collect the current **read-only** AWS API calls available and execute a brute-force attack against each call. This will allow the Operator to determine which permissions exist. Knowing these calls can assist in finding the services to target further.

## Installation

Generally the instructions provided [here](https://github.com/andresriancho/enumerate-iam#installation) are sufficient. The commands executed below are strictly for personal preference.

```bash
git clone git@github.com:andresriancho/enumerate-iam.git
cd enumerate-iam/
python3 -m venv .
source bin/activate
pip install -r requirements.txt
```

## Modification

Follow the instructions provided in the [original GitHub](https://github.com/andresriancho/enumerate-iam?tab=readme-ov-file#updating-the-api-calls) to pull the current AWS calls. Once that is complete, the files can be modified to only target specific services.

Your current directory should be `.../enumerate-iam/enumerate-iam` with the following files:

![`ls` in `enumerate-iam` directory](images/ls_cmd.png)

The file which can be modified to specify tests is `bruteforce_tests.py`. 

Open the file and remove the portions which are associated with services that should not be targeted. The file can then be saved and the tool executed as normal.

For example, if IAM and EC2 actions are the only service which should be checked for, the content of `bruteforce_tests.py` would contain the following:

```python
BRUTEFORCE_TESTS = {
    "iam": [
        "get_account_authorization_details",
        "get_account_password_policy",
        "get_account_summary",
        "get_credential_report",
        "get_user",
        "list_access_keys",
        "list_account_aliases",
        "list_groups",
        "list_instance_profiles",
        "list_mfa_devices",
        "list_open_id_connect_providers",
        "list_policies",
        "list_roles",
        "list_saml_providers",
        "list_server_certificates",
        "list_service_specific_credentials",
        "list_signing_certificates",
        "list_ssh_public_keys",
        "list_users",
        "list_virtual_mfa_devices"
    ],
    "ec2": [
        "describe_account_attributes",
        "describe_address_transfers",
        "describe_addresses",
        "describe_addresses_attribute",
        "describe_aggregate_id_format",
        "describe_availability_zones",
        "describe_aws_network_performance_metric_subscriptions",
        "describe_bundle_tasks",
        "describe_capacity_reservation_fleets",
        "describe_capacity_reservations",
        "describe_carrier_gateways",
        "describe_classic_link_instances",
        "describe_client_vpn_endpoints",
        "describe_coip_pools",
        "describe_conversion_tasks",
        "describe_customer_gateways",
        "describe_dhcp_options",
        "describe_egress_only_internet_gateways",
        "describe_elastic_gpus",
        "describe_export_image_tasks",
        "describe_export_tasks",
        "describe_fast_launch_images",
        "describe_fast_snapshot_restores",
        "describe_fleets",
        "describe_flow_logs",
        "describe_fpga_images",
        "describe_host_reservation_offerings",
        "describe_host_reservations",
        "describe_hosts",
        "describe_iam_instance_profile_associations",
        "describe_id_format",
        "describe_images",
        "describe_import_image_tasks",
        "describe_import_snapshot_tasks",
        "describe_instance_connect_endpoints",
        "describe_instance_credit_specifications",
        "describe_instance_event_notification_attributes",
        "describe_instance_event_windows",
        "describe_instance_status",
        "describe_instance_topology",
        "describe_instance_type_offerings",
        "describe_instance_types",
        "describe_instances",
        "describe_internet_gateways",
        "describe_ipam_byoasn",
        "describe_ipam_external_resource_verification_tokens",
        "describe_ipam_pools",
        "describe_ipam_resource_discoveries",
        "describe_ipam_resource_discovery_associations",
        "describe_ipam_scopes",
        "describe_ipams",
        "describe_ipv6_pools",
        "describe_key_pairs",
        "describe_launch_template_versions",
        "describe_launch_templates",
        "describe_local_gateway_route_table_virtual_interface_group_associations",
        "describe_local_gateway_route_table_vpc_associations",
        "describe_local_gateway_route_tables",
        "describe_local_gateway_virtual_interface_groups",
        "describe_local_gateway_virtual_interfaces",
        "describe_local_gateways",
        "describe_locked_snapshots",
        "describe_mac_hosts",
        "describe_managed_prefix_lists",
        "describe_moving_addresses",
        "describe_nat_gateways",
        "describe_network_acls",
        "describe_network_insights_access_scope_analyses",
        "describe_network_insights_access_scopes",
        "describe_network_insights_analyses",
        "describe_network_insights_paths",
        "describe_network_interface_permissions",
        "describe_network_interfaces",
        "describe_placement_groups",
        "describe_prefix_lists",
        "describe_principal_id_format",
        "describe_public_ipv4_pools",
        "describe_regions",
        "describe_replace_root_volume_tasks",
        "describe_reserved_instances",
        "describe_reserved_instances_listings",
        "describe_reserved_instances_modifications",
        "describe_reserved_instances_offerings",
        "describe_route_tables",
        "describe_scheduled_instances",
        "describe_security_group_rules",
        "describe_security_groups",
        "describe_snapshot_tier_status",
        "describe_snapshots",
        "describe_spot_datafeed_subscription",
        "describe_spot_fleet_requests",
        "describe_spot_instance_requests",
        "describe_spot_price_history",
        "describe_store_image_tasks",
        "describe_subnets",
        "describe_tags",
        "describe_traffic_mirror_filter_rules",
        "describe_traffic_mirror_filters",
        "describe_traffic_mirror_sessions",
        "describe_traffic_mirror_targets",
        "describe_transit_gateway_attachments",
        "describe_transit_gateway_connect_peers",
        "describe_transit_gateway_connects",
        "describe_transit_gateway_multicast_domains",
        "describe_transit_gateway_peering_attachments",
        "describe_transit_gateway_policy_tables",
        "describe_transit_gateway_route_table_announcements",
        "describe_transit_gateway_route_tables",
        "describe_transit_gateway_vpc_attachments",
        "describe_transit_gateways",
        "describe_trunk_interface_associations",
        "describe_verified_access_endpoints",
        "describe_verified_access_groups",
        "describe_verified_access_instance_logging_configurations",
        "describe_verified_access_instances",
        "describe_verified_access_trust_providers",
        "describe_volume_status",
        "describe_volumes",
        "describe_volumes_modifications",
        "describe_vpc_classic_link",
        "describe_vpc_classic_link_dns_support",
        "describe_vpc_endpoint_connection_notifications",
        "describe_vpc_endpoint_connections",
        "describe_vpc_endpoint_service_configurations",
        "describe_vpc_endpoint_services",
        "describe_vpc_endpoints",
        "describe_vpc_peering_connections",
        "describe_vpcs",
        "describe_vpn_connections",
        "describe_vpn_gateways",
        "get_aws_network_performance_data",
        "get_ebs_default_kms_key_id",
        "get_ebs_encryption_by_default",
        "get_image_block_public_access_state",
        "get_instance_metadata_defaults",
        "get_serial_console_access_status",
        "get_snapshot_block_public_access_state",
        "get_vpn_connection_device_types",
        "list_images_in_recycle_bin",
        "list_snapshots_in_recycle_bin"
    ]
}
```

## Conclusion

This modification reduces the time to execute scans. Furthermore, it is recommend to adjust the tool as described in the pull request [here](https://github.com/andresriancho/enumerate-iam/issues/14#issuecomment-1079720593).

The fix is to modify `max_attempts` to `3` in the file `main.py`.

Hope this was helpful! Thank you for your time.

[Back to Home](https://blog.the1ntern.net)